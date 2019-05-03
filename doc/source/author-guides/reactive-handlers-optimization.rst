.. _reactive-handlers-optimization:

=======================================
Optimizing Reactive Handlers in a Charm
=======================================

One of the issues often encountered when writing reactive charms is the
disconnect between the ``hook`` that is invoked, and the handlers that actually
run.  This article explores that relationship, or lack of relationship, and how
the side-effects can be mitigated.

Introduction
~~~~~~~~~~~~

The charms.reactive library supports ``@hook(...)`` handlers which respond to
*actual* Juju hook invocations and ``@when(...)`` state-type handlers that are
controlled by combinations of ``states`` (which are essentially boolean flags).

*Note: that 'states' are being renamed to 'flags' to better reflect their usage
as true/false condition checks for the @when(...) type handlers.  In this
article the term 'state/flag' means the string from set_state(...) or
remove_state(...).*

Hook handlers run before any state handlers.  Hooks *can't* be combined with
state/flag handlers.  The state handlers then run until there are no more state
changes.

The can cause unexpected behavior as it means that state handlers are run
whenever their condition state/flags evaluate to 'true' for *any* hook that
runs.

Example
~~~~~~~

Let's say that a charm needs a database interface (say ``shared-db``) and when
it connects the charm sends the username/database the it wants to connect to,
and then when it's available, then ensures (via some sync mechanism) that the
database tables are set up.  So the sequence of events is:

1. When ``shared-db.connected``: send the credentials.
2. When ``shared-db.available``: sync the database.

The *interface* is usually written in such a way that:

a) When the relation hook ``some-relation-joined`` is handled, the
   ``<name>.connected`` state/flag is set.
b) When the relation hook ``some-relation-changed`` is handled, the data that
   is on that relation from the remote party is checked/validated in some way, and
   if it is *usable*, as defined by the interface, then the ``<name>.available``
   state/flag is set.
c) When either ``some-relation-departed`` or ``some-relation-broken`` is
   handled, then both ``<name>.available`` and ``<name>.connected`` are removed.

Then, typically the handler code in the charm which uses that interface looks
something like this:

.. code-block:: python

        @when_not('installed')
        def install():
            # do installation
            set_state('installed')


        @when('<name>.connected')
        def do_connection(shared_db):
            shared_db.send('username', 'database')


        @when('<name>.available')
        def do_sync(shared_db):
            # do some database sync.


The Issue:
~~~~~~~~~~

The implementation for the *interface* is typical.  If you look at the
`interfaces.juju.solutions`_ and pick almost any interface (e.g.
`interface-etcd`_) you'll see that the various ``...connected`` and
``..available`` state/flags are *continuous* from the moment the conditions for
them are met.

.. _`interfaces.juju.solutions`: https://github.com/juju-solutions/
.. _`interface-etcd`: https://github.com/juju-solutions/interface-etcd

By now you will have probably seen the issue; once the ``<name>.connected``
state/flag is set, the ``do_connection(shared_db)`` function will be called
for *ANY* subsequent hook, and that includes the ``update-status`` hook which
runs, by default, every 5 minutes.

This would mean that the ``shared_db.send(...)`` function will be called every
5 minutes, *possibly* sending data to the connected database charm and causing
it to do work, due to the relevant *changed* hook being called.

The same is true for the ``<name>.available`` state/flag; it will cause the
``do_sync(...)`` handler to be called for every subsequent hook.

I term this: **Doing too much work in update-status**.

Solutions:
~~~~~~~~~~

There are a couple of ways of solving this.  One works extremely well for the
simpler example shown here, and the other can be used for the more complicated
case when data on the interface may change that *needs to be handled*.

Option 1: Gating handlers with a *done* state
---------------------------------------------

If we re-write our initial code block as:

.. code-block:: python

        @when_not('installed')
        def install():
            # do installation
            set_state('installed')
            remove_state('shared-db.details.sent')
            remove_state('shared-db.synced')


        @when('<name>.connected')
        @when_not('shared-db.details.sent')
        def do_connection(shared_db):
            shared_db.send('username', 'database')
            set_state('shared-db.details.sent')


        @when('<name>.available')
        @when_not('shared-db.synced')
        def do_sync(shared_db):
            # do some database sync.
            set_state('shared-db.synced')

Now we have *run once* handlers that *can* be run again if the author of
the charm wishes to, unlike the ``@only_once`` decorator which will *only ever*
run that handler once, which sometimes may be useful.

So, for example, if the charm were to be upgraded, the ``upgrade-charm`` hook
could be used to clear the ``installed`` state, thus allowing the charm to
upgrade the installed software and then run the ``do_connection(...)`` and
``do_sync(...)`` handlers another time.

Option 2: Checking for data changes
===================================

The other method is to check the interfaces for data changes.  This can be done
in two ways:

1) In the interface, when the ``<name>-relation-changed`` hook is handled, see
   if the data changed, and set a ``<name>.changed`` state, that is then cleared
   after the all the handlers have run for that hook - this is achieved using an
   ``atexit(...)`` function.

2) In the charm layer code, use a data change detection function to decide if
   the handler should be run.

Note that re-writing an interface may not be an option as other charms may
still be dependent on that interface's functionality.  Thus, often, only the
2nd method can be employed.

Option 2.1: Re-writing the interface
------------------------------------

A typical interface may take the following form (this is the ``requires.py`` side):

.. code-block:: python

        class SomeClient(RelationBase):
            scope = scope.GLOBAL

            @hook('{requires:name}-relation-{joined,changed}')
            def changed(self):
                self.set_state('{relation_name}.connected')
                # if we have some piece of data
                if self.get_data():
                    self.set_state('{relation_name}.available')
                else:
                    self.remove_state('{relation_name}.available')

            @hook('{requires:name}-relation-{broken,departed}')
            def gone_away(self):
                self.remove_state('{relation_name}.connected')
                self.remove_state('{relation_name}.available')

            def get_data(self):
                 return self.get_remote('some-data-item')

And in the charm layer, using this interface:

.. code-block:: python

        @when('<name>.available')
        def do_something_with_data(the_if_object):
            do_something_with(the_if_object.get_data())


As mentioned above, a very typical style for an interface.  In order to
implement the *data-changed* idea, we can use the
``charms.reactive.helpers.data_changed()`` function like this:

.. code-block:: python

        import charms.reactive.helpers as helpers
        import charmhelpers.core.hookenv as hookenv


        class SomeClient(RelationBase):
            scope = scope.GLOBAL

            @hook('{requires:name}-relation-{joined,changed}')
            def changed(self):
                self.set_state('{relation_name}.connected')
                # if we have some piece of data
                data = self.get_data()
                if data:
                    self.set_state('{relation_name}.available')
                    if helpers.data_changed('interface-name.get_data', data):
                        self.set_state('{relation_name}.changed')
                        hookenv.atexit(
                            lambda: self.remove_state('{relation_name}.changed'))
                else:
                    self.remove_state('{relation_name}.available')

            @hook('{requires:name}-relation-{broken,departed}')
            def gone_away(self):
                self.remove_state('{relation_name}.connected')
                self.remove_state('{relation_name}.available')

            def get_data(self):
                 return self.get_remote('some-data-item')

Using the ``<name>.changed`` state can either be simple or a bit more
complicated depending on whether multiple handlers need to see the state.  The
issue here is that the ``<name>.changed`` state is *transitory*, whereas we
would want the charm to recover from errors as much as possible, and thus want
to physically clear the state in the charm.  The way to do that is to use a
secondary state as the trigger to do the work needed. e.g.:

.. code-block:: python

        @when('<name>.changed')
        def name_has_changed(*args):
            set_state('name_has_changed')

        @when('name_has_changed')
        @when('<name>.available')
        def do_something_with_data(the_if_object):
            do_something_with(the_if_object.get_data())
            remove_state('name_has_changed')


The slight increase in complexity allows the ``<name>.changed`` state/flag to
be used for several handlers, with each handler having its own guard
state/flag.  It also means that if the charm code were to fail during an
invocation that the ``name_has_changed`` state would *still* indicate that the
data had changed and thus on the *next* invocation of the charm the handler
would still be called.

Note that modifying an existing interface in this way doesn't affect the
functionality of existing charm layers which don't 'know' about a
``<name>.changed`` state/flag.  They would continue to function as previously.

The debug logs will show that the ``name_has_changed`` handler will run,
followed by the ``do_something_with_data`` at some later stage *in the same
hook invocation*.  If no data has changed, then neither of these handlers will
be called, leading to a cleaner debug-log which reflects what has actually been
used/run in the charm.

There is a small window of failure possible where the charm may crash before
the ``name_has_changed`` handler has had a chance to run.  If this concerns you
then Option 2.2 below may be more suited.

Option 2.2: Use change detection in the charm
---------------------------------------------

An alternative approach is to *only* do data changed detection in the charm
layer.  I recommend *NOT* using the ``data_changed()`` function from the
``charms.reactive`` library as it can only be called *once* for each time data
is changed.  i.e. if ``data_changed(...)`` is called for the same data key and
data more than once, it will only return ``True`` for the first call, and then
``False`` thereafter.

This is bad because if the charm code fails/crashes *after* calling
``data_changed(...)`` then on the *next* invocation of the code the data
*won't* appear to be changed and the original intent of the handler won't be
honored.  i.e. the charm will fail to use the changed data.

So the  ``charms.openstack`` library supports a slightly different version
called ``is_data_changed(..)`` which works as a context manager, and doesn't
*change* the stored data until the context scope is exited without an
Exception.  It can be used as follows:

.. code-block:: python

        import charms_openstack.charm.utils as utils

        @when('<name>.available')
        def do_something_with_data(the_if_object):
            data = the_if_object.get_data()
            with utils.is_data_changed('some-meaningful-key', data) as f:
                if f:
                    do_something_with(data)


The extra ``f`` is slightly awkward, but it's how the code can discover that
the data has changed.  Also, note that this version is *less efficient* than
the 'changed-interface' version as every handler needs to use
``is_data_changed(...)`` which has an overhead.  Also, the debug logs from Juju
will show that every handler is still being called, so you lose some context
information about which handlers are actually doing work.

Summary
~~~~~~~

This article has shown some of the pitfalls around ``charms.reactive`` and
handlers and how handlers can inadvertently cause too much work to be done in
either connected charms, or the software payload being managed.

The two options described provide some tools to the charm author to reduce
workloads during hooks and provide a cleaner, more understandable debug-log.
They also mitigate against unexpected side-effects in charms where handlers the
author *thinks* might only run once, in fact run every time the charm code is
invoked via a hook and thus may cause unnecessary, redundant or, worst, bugs
in either the payload, or other connected charms.
