.. _coding_guidelines:

=================
Coding Guidelines
=================

Introduction
------------

We write code for OpenStack charms.  Mostly in Python.  They say that code is
read roughly 20 times more than writing it, and that's just the process of
writing code.  Reviewing code and modifying it means that it will be read many,
many times.  Let's make it as easy as possible.  We're lucky(!) with Python as
the syntax ensures that it roughly always looks the same.

As OpenStack charms are for OpenStack it's a good idea to adhere to the
OpenStack Python coding standard.  So first things first:

* Read the `OpenStack Coding standard <https://docs.openstack.org/hacking/latest/>`__.
* Read PEP8 (again).

Topics
------

Multiple roots -- symlinks
~~~~~~~~~~~~~~~~~~~~~~~~~~

Multiple roots with symlinks create issues in charms.  This is where, for
example, charmhelpers is symlinked into both a hooks and actions subdirectory.
This creates a situation where the *same* modules are loaded into the Python
interpreter's memory twice at two different module paths.  This creates
problems with testing as *depending on the load time ordering* it might not be
clear which particular module path you're trying to mock out and which one is
first in the module path map.

So only every have ONE root for your python code in a charm.  e.g. put it in
``/lib`` and add that to path by ``sys.path.append('lib').``

Incidentally, if you **are** mocking out code in charmhelpers in you charms,
**it's probably not a good idea**.  Only mock code in the target object file,
rather than an included module.

Install-time vs Load-time vs Runtime code
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The hooks in charms are effectively short-term running scripts.  However,
despite being short-lived, the code invoked is often complex with multiple
modules being imported which also import other modules.

It's important to be clear on what is *load time* code and _runtime_ code.
Although there is no actual distinction in Python, it's useful to think of
*runtime* starting when the following code is reached:

.. code:: python

    if __name__ == '__main__':
        do_something()

I.e. the code execution of ``do_something()`` is runtime, with everything
preceding being loadtime.

So why is the distinction useful?  Put simply, *it's much harder to test*
load-time code in comparison to runtime code with respect to mocking.  Consider
these two fragments:

**Bad:**

.. code:: python

    import a.something

    OUR_CONFIG = {
        'some_thing': a.something.config('a-value'),
    }

**Good:**

.. code:: python

    import a.something


    def get_our_config():
        return {
            'some_thing': a.something.config('a-value'),
        }

If performance is an issue (i.e. multiple calls to ``config()`` are expensive)
then either use a ``@caching`` type decorator, or just doing it manually. e.g.

.. code:: python

    _our_config = None

    def get_our_config():
        if _our_config is None:
            _our_config = {
                'some_thing': a.something.config('a-value'),
            }
        return _our_config

In the bad example, in order to mock out the config module we have to do
something like:

.. code:: python

    with patch('a.something.config') as mock_config:
        import a.something.config

This also relies on this being the _first_ time that module has been imported.
Otherwise, the module is already cached and config can't be mocked out.

Compare this with the good example.

.. code:: python

    def test_out_config(self):
        with patch('module.a.something.config') as mock_config:
            mock_config.return_value = 'thing'
            x = model.get_out_config()

This brings us to:

CONSTANTS should be simple
~~~~~~~~~~~~~~~~~~~~~~~~~~

In the bad example above, the constant ``OUR_CONFIG`` is defined as load-time by
calling ``a.something.config()``.  Thus, in reality, the constant is being
defined at load-time using a runtime function that returns a value - it's
dynamic.

Don't:

.. code:: python

    CONFIG = {
        'some_key': config('something'),
    }

This is actually a *function in disguise*.

Prefer:

.. code:: python

    def get_config():
        return {
            'some_key': config('something'),
        }

Why?

So that you can mock out ``get_config()`` or ``config()`` at the test run time,
rather than before the module loads.  This makes testing easier, more
predictable, and also makes it obvious that it's not really a constant, but
actually a function which returns a structure that is dynamically generated
from configuration.

And **definitely** don't do this at the top level in a file:

.. code:: python

    CONFIGS = register_configs()

You've just created a load time test problem _and_ created a CONSTANT that
isn't really one.  Just use ``register_configs()`` directly in the code and write
``register_configs()`` to be ``@cached`` if performance is an issue.


Decorators
~~~~~~~~~~

There shouldn't be much need to write a decorator.  They definitely **should
not** be used instead of function application or instead of context managers.
When they are used it's preferable that they are orthogonal to the function
they are decorating, and don't change the nature of the function.

functools.wraps(f)
++++++++++++++++++

If they are used, then they should definitely make use of ``functools.wraps`` to
preserve the function name of the original function and it's docstring.  This
makes stacktraces more readable.  e.g.:

.. code:: python

    def my_decorator(f):
        functools.wraps(f):
        def decoration(*args, **kwargs):
            # do soemthing before the function call?
            r = f(*args, **kwargs)
            # do soemthing after the function call?
            return r

        return decoration

Mocking out decorators
++++++++++++++++++++++

If the decorator's functionality is orthogonal to the function, then mocking
out the decorator shouldn't be necessary.  However, if it *isn't* then tweaking
how the decorator is written can make it easier to mock out the decorator.

Consider the following code:

.. code:: python

    @a_decorator("Hello")
    def some_function():
        pass

    def a_decorator(name):
        def outer(f):
            @functools.wraps(f)
            def inner(*args, **kwargs):
                # do something before the function
                r = f(*args, **kwargs)
                # do something after the function
                return r

            return inner

        return outer

It's very difficult to test some_function without invoking the decorator, and
equally, it's difficult to stop the decorator from being applied to the
function without mocking out ``@a_decorator`` before importing the module under
test.

However, with a little tweaking of the decorator we can mock out the decorator
without having to jump through hoops:

.. code:: python

    def a_decorator(name):
        def outer(f):
            @functools.wraps(f)
            def inner(*args, **kwargs):
                return _inner(name, args, kwargs)
            return inner
        return outer

    def _inner(name, args, kwargs):
        # do something before the function
        r = f(*args, **kwargs)
        # do something afterwards
        return r

Now, we can easily mock ``_inner()`` after the module has been loaded, thus
changing the function of the decorator _after_ it has been applied.


Import ordering and style
~~~~~~~~~~~~~~~~~~~~~~~~~

Let's be consistent and ensure that we have the same import ordering and style
across all of the charms (and other code) that we release.

Use absolute imports
++++++++++++++++++++

Use absolute imports.  In Python 2 code this means also that we should force
absolute imports:

.. code:: python

    from __future__ import absolute_import

We should use absolute imports so that we don't run into module name clashes
across our own modules, nor with system and 3rd party packages.  See
https://www.python.org/dev/peps/pep-0328/#id8 for more details.


Import ordering
+++++++++++++++

* Core Python system packages
* Third party modules
* Local modules

They should be alphabetical order, with a single space between them, and
preferably in alphabetical order.  If load order is important (and it shouldn't
be!) then that's the only reason they shouldn't be in alpha order.

Import Style
++++++++++++

It's preferable to import a module rather than an object, class, function or
instance from a module.

Prefer:

.. code:: python

    import module

    module.function()

over:

.. code:: python

    from module import function

    function()

However, if there are good reasons to import from a module, and there is more
than one item, then the style is:

.. code:: python

    from module import (
        one_import_per_line,
    )

Why?

Using ``import module; module.function()`` rather than ``from module import
function`` is preferable because:

* with multiple imports, more symbols are being brought into the importing
  modules namespace.
* It's clearer in the code when an external function is being used, as it is
  always prefixed by the external module name.  This is useful as it makes it
  more obvious what is happening in the code.

Only patch mocks in the file/module under test
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

A unit test often needs to mock out functions, classes or instances in the file
under test.  The mocks should _only_ be applied to the file that contains the
item that is being tested.

Don't:

.. code:: python

    # object.py
    import something

    def function_under_test(x):
        return something.doing(x)

In the unit test file: ``test_unit.py``:

.. code:: python

    # test_unit.py
    def unit_test():
        with patch('something.doing') as y:
            y.return_value = 5
            assert function_under_test(3) == 5

Prefer:

.. code:: python

    # object.py
    import something

    def function_under_test(x):
        return something.doing(x)

In the unit test file: ``test_unit.py``:

.. code:: python

    # test_unit.py
    def unit_test():
        with patch('object.something.doing') as y:
            y.return_value = 5
            assert function_under_test(3) == 5

i.e. the thing that is patched is in object.py **not** in the library file
'something.py'

Don't use _underscore_methods outside of the class
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Underscore methods are supposed to be, by convention, private to the enclosing
scope, be that a module or a class.  They are used to signal that the method is
_private_ even though the privacy can't be enforced.

Thus don't do this:

.. code:: python

    class A():
        def _private_method():
            pass

    x = A()
    x._private_method()

Simply rename the method without the underscore.  Otherwise you break the
convention and people will not understand how you are using *private methods*.

Equally, don't use them in derived classes _either_.  A private method is
supposed to be private to the class, and not used in derived classes.

Only use list comprehensions when you want the list
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Don't:

.. code:: python

    [do_something_with(thing) for thing in mylist]

Prefer:

.. code:: python

    for thing in mylist:
        do_something_with(thing)

Why?

You just created a list and then threw it away.  And it's actually less clear
what you are doing.  Do use list comprehensions when you actually want a list
to do something with.

Avoid C-style dictionary access in loops
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Don't:

.. code:: python

    for key in dictionary:
        do_something_with(key, dictionary[key])

Prefer:

.. code:: python

    for key, value in dictionary.items():
        do_something_with(key, value)

Why?

Using a list of keys to access a dictionary is less efficient and less obvious
as to what's happening.  ``key, value`` could actually be ``config_name`` and
``config_item`` which means the code is more self-documenting.

Also remember that ``dictionary.keys()`` & ``dictionary.values()`` exist if you
want to explicitly iterate just over the keys or values of a dictionary.  Also,
it's preferable to iterate of ``dictionary.keys()`` rather than ``dictionary``
because, whilst they do the same thing, it's not as obvious what is happening.

If performance is an issue (Python2) then ``iterkeys()`` and ``itervalues()`` for
generators, which is the default on Python3.

Prefer tuples to lists
~~~~~~~~~~~~~~~~~~~~~~

Tuples are non malleable lists, and should be used where the list isn't going
to change.  They have (slight) performance advantages, but come with a
guarantee that the list won't change - note the objects within the tuple could
change, just not their position or reference.

Thus don't:

.. code:: python

    if x in ['hello', 'there']:
        do_something()

Prefer:

.. code:: python

    if x in ('hello', 'there'):
        do_something()

However, remember the caveat.  A single item tuple literal  has to have a
trailing comma:

.. code:: python

    my_tuple = ('item', )


Prefer CONSTANTS to string literals or numbers
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This is the "No magic numbers" rule. In a lot of the OS charms there is code
like:

.. code:: python

    db = kv()
    previous_thing = db.get('thing_key', thing)

Prefer:

.. code:: python

    THING_KEY = 'thing_key'

    db = kv()
    previous_thing = db.get(THING_KEY, thing)

Why?

String literals introduce a vector for mistakes.  We can't use the language to
help prevent spelling mistakes, nor our tools to do autocompletion, nor use
lint to find 'undefined' variables.  This also means that if you use the same
number or string literal more than once in code you should create a constant
for that value and use that in code.  This includes fixed array accesses,
offsets, etc.

Don't abuse __call__()
~~~~~~~~~~~~~~~~~~~~~~

``__call__()`` is a method that is invoked when ``()`` is invoked on an object --
``()`` on a class invokes ``__call__`` on the metaclass for the class.

A good example of abuse of ``__call__`` is the class ``HookData()`` which, to
access the context manager, is invoked as:

.. code:: python

    with HookData()() as hd:
        hd.kv.set(...)

The sequence ``()()`` is almost certainly a *code smell*.  There is hidden
behaviour that requires you to go to the class to see what is actually
happening.   It would have been more obvious if that method was just called
``cm()`` or ``context()``:


.. code:: python

    with HookData().context() as hd:
        hd.kv.set(...)


Don't use old style string interpolation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code:: python

    action_fail("Cannot remove service: %s" % service.host)

Prefer:

.. code:: python

    action_fail("Cannot remove service: {}".format(service.host))

Why?

It's the new style, and the old style is deprecated; eventually it will be
removed.  Plus the new style is way more powerful: keywords, dictionary
support, to name but a few.

Docstrings and comments
~~~~~~~~~~~~~~~~~~~~~~~

Docstrings and comments are there to inform a reader of the code additional,
contextual, information that isn't readily available by just reading the code.
Docstrings can also be used to automatically generate *useful* documentation
for programmers who are using those functions.  This is particularly important
in the case of a library, but is also very important simply from a maintenance
perspective.  Being able to look at the docstring for a function and quickly
understand the types of the parameters and the return type helps to understand
the code *much more quickly* than hunting through other code trying to
understand what types of things might be sent to the function.

In futher, types in docstrings will become part of the *linting* of the code
(as part of PEP8) and so, good practice now, will help with more maintainable
code in the future.

Comments are important to help the reader of the code understand what is being
implemented, rather than just repeating what the code does.  A good comment is
minimal and terse, yet still explains the purpose behind a segment of code.

Docstring formats are slightly complicated by whether we are doing Python 2
code, Python 3 code, or a shared library.  For Python 2 and Python 2 AND 3
compatible code (e.g. charm-helpers) there is a preferred approach, and for
Python 3 only code there is a separate preferred approach.

Python 2 code and Python 2/3 compatible code
--------------------------------------------

Python 2 compatible code docstrings are constrained by not being able to have
mypy_ annotations in the code.  We don't really want to add mypy annotations
into comments, so we've adopted a docstring convention which informs as to what
the types are, without being able to actually statically check it.

The main reason for *not* using mypy compatible comments is that they are
fairly ugly.  As we are not using, nor plan to use, mypy_ on Python 2 code, we
can do something that is a little more aesthetically pleasing.

Every function exported by a module should have a docstring.  Generally, this
means all functions mentioned in ``__ALL__`` or implicitly those that do not
start with an ``_``.

The preferred format for documenting parameters and return values is
ReStructuredText (reST) as described: http://docutils.sourceforge.net/rst.html
but with mypy type signatures.  Classes will use the ``:class:`ClassName```
type declaration so that sphinx can appropriately underline when using autodoc.

The field lists are described here:
http://www.sphinx-doc.org/en/stable/domains.html#info-field-lists

An example of an acceptable function docstring is:

.. code:: python

    def mult(a, b):
        """Multiple a * b and return the result.

        :param a: Number
        :type: Union[int, float]
        :param b: Number
        :type: Union[int, float]
        :returns a * b
        :rtype: Union[int, float]
        :raises: ValueError, TypeError if the params are not numbers
        """
        return a * b


    def some_function(a):
        """Do something with the FineObject a

        :param a: a fine object
        :type: :class:`FineObject`
        """
        do_something_with(a)


Other comments should be used to support the code, but not just re-say what the
code is doing.

Python 3 code
-------------

The situation is a little more complicated for Python 3 code. Ideally, we would
just use Python 3.6 mypy_ annotations, but Xenial *only* has Python 3.5.  This
means that some types of annotations aren't possible.  As Xenial is supported
until 2021, until that time, all Python 3 mypy_ annotations will need to be
supported on Python 3.5.

This means that PEP-526 can't be used (Syntax for variable annotations) and
PEP-525 (Asynchronous generators) and PEP-530 (comprehensions) are also not
possible.

So the minimal preferred docstring format for Python 3 code is the same as
Python 2.  However, ideally, mypy_ notations will be used:

.. code:: python

    def mult(a: Union[int, float],
             b: Union[int, float]) -> Union[int, float]:
        """Multiple a * b and return the result"""
        return a * b


    def some_function(a: FineObject):
        """Do something with a FineObject

        :param: a is used in the context of doing something.
        """
        do_something_with(a)

.. note::

    Because mypy annotations tell you what the types are and this type
    information can be checked statically, it means that we don't have to
    specify what the function might raise as an exception, as that would be a
    type error.  e.g. if at runtime the function ``mult(...)`` was supplied
    with an object that had no ``*`` implementation, then the code would raise
    an exception.  However, linting on fully typed code would prevent this.
    Hence we don't, for function ``mult`` need to provide either a return type
    in the docstring, nor a ``:raises:`` line.

    In the ``some_function(...)`` we have optionally specified the ``:param:``
    to provide additional information to the docstring for the user.  The type
    will be provided by ``sphinx`` autodoc.

The end objective with the Python 3 code is to use mypy_ (or pyre_) to
statically check the code in the CI server prior to check-ins.


.. _mypy: http://mypy-lang.org/
.. _pyre: https://pyre-check.org/




Ensure there's a comma on the last item of a dictionary
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This helps when the developer adds an item to a dictionary literal, in that
they don't have to edit the previous line to add a comma.  It also means that
the review doesn't indicate that the previous line has changed (due to the
addition of a comma).

Prefer:

.. code:: python

    a_dict = {
        'one': 1,
        'two': 2,
    }

over:

.. code:: python

    a_dict = {
        'one': 1,
        'two': 2
    }

Avoid dynamic default arguments in functions
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Don't use a dynamic assignment to a default argument.  e.g.

.. code:: python

    def a(b=[]):
        b.append('hello')
        print b

    In [2]: a()
    ['hello']

    In [3]: a()
    ['hello', 'hello']

As you can see, the list is only assigned the first time, and thereafter it
'remember' the previous values.

Also avoid other default, dynamic, assignments:

.. code:: python

    def f():
        return ['Hello']


    def a(b=f()):
        b.append('there')
        print b


    In [3]: a()
    ['Hello', 'there']

    In [4]: a()
    ['Hello', 'there', 'there']

Instead, prefer:


.. code:: python

    def a(b=None):
        if b is None:
            b = f()
        b.append('there')
        print b


    In [6]: a()
    ['Hello', 'there']

    In [7]: a()
    ['Hello', 'there']

Why?

Although it can be a handy side-effect for allowing a function to remember
previous values, due to a quirk in the interpreter in only assigning the
reference once, it may be changed in the future and it hides the intention of
the code.

Avoid side effects in Adapters and Contexts
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Adapters (reactive charms) and Contexts should not alter the unit they are
running, i.e. should not have unexpected side effects. Some environment
altering side effects do exist in older contexts, however this should not be
taken as an indicator that it is acceptable to add more.

Why?

Adapters and Contexts are regulary called via the update status hook to assess
whether a charm is ready. If calling the Context or Adapter has unexpected
side effects it could interrupt service. See `Bug #1605184 <https://bugs.launchpad.net/charms/+source/nova-cloud-controller/+bug/1605184>`__ for an example of this issue.
