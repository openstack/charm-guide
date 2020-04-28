.. _making-a-change:

Making a change
===============

So you've found a bug, or want to implement a new feature.  OpenStack charms
use git-gerrit to control the addition (committing changes) to the various
repositories.  Outlined below is the *general* approach to making changes.

There are two different architectural approaches to the OpenStack charms:
*charm-helpers charms*, and *charms.openstack reactive, layered, charms*.
charm-helpers-style charms have an extra *gotcha* when it comes to syncing new
versions of charm-helpers to the charm.  Please see the note at the end.

Development Workflow
~~~~~~~~~~~~~~~~~~~~

Broadly the work flow for making a change to a charm is:

.. code:: bash

    git clone https://opendev.org/openstack/charm-cinder
    cd charm-cinder
    git checkout -b bug/XXXXXX master

Make the changes you need within the charm, and then run unit and pep8 tests using tox:

.. code:: bash

    tox

resolve any problems this throws up, and then commit your changes:

.. code:: bash

    git add .
    git commit

Commit messages should have an appropriate title, and more detail in the body; they
can also refer to bugs:

.. code:: bash

    Closes-Bug: #######
    Partial-Bug: #######
    Related-Bug: #######

Gerrit will automatically link your proposal to the bug reports on launchpad and
mark them fix committed when changes are merged.

Execute pep8 and unit tests:

.. code:: bash

    tox

Finally, submit your change for review (if they pass pep8 and unit tests!):

.. code:: bash

    git review

This will push your proposed changes to Gerrit and provide you with a URL for the
review board on https://review.opendev.org/.

To make amendments to your proposed change, update your local branch and then:

.. code:: bash

    git commit --amend
    git review

Functional test changes
~~~~~~~~~~~~~~~~~~~~~~~

When writing new functional tests with Zaza, it is often necessary to run
the functional tests with changes to the tests not yet landed. It is therefore
possible to run the tests in CI and pull in an open GitHub Pull Request. This
is accomplished via specific text in the commit message, specifically:
``func-test-pr:``. The command is not case sensitive, and should contain a
link to a GitHub PR, for example:

.. code:: bash

   func-test-pr: https://github.com/openstack-charmers/zaza-openstack-tests/pull/5

When OpenStack Charm CI runs the functional gate on a commit with the above in
its message, the git ref that the above PR references will be
substituted in place of the zaza-openstack package that is in the charm's
``test-requirements.txt``.

Stable charm updates
~~~~~~~~~~~~~~~~~~~~

Any update to a stable charm must first be applied into the master branch; it should
then be cherry-picked in a review for the stable branch corresponding to your target
release (ensuring that any interim releases have the fix landed):

.. code:: bash

    git checkout -b stable/bug/XXXX origin/stable/YYYY
    git cherry-pick -x <hash of master branch commit>
    git review

Where XXXX is the launchpad bug ID corresponding to the fix you want to backport and
YYYY is the release name you are targeting e.g. 16.04

.. note:: when cherry-picking a commit and/or modifying the commit message, always ensure that
          the original Change-Id is left intact.

charm-helpers style charms
~~~~~~~~~~~~~~~~~~~~~~~~~~

In a charm-helpers style charm, **charm-helpers** is synced into the charm using
a *make* command.  Inspecting the ``Makefile`` of, say, `charm-keystone
<https://opendev.org/openstack/charm-keystone>`_ shows:

.. code:: Makefile

    sync: bin/charm_helpers_sync.py
        @$(PYTHON) bin/charm_helpers_sync.py -c charm-helpers-hooks.yaml
        @$(PYTHON) bin/charm_helpers_sync.py -c charm-helpers-tests.yaml

This command takes code from the `charm-helpers
<https://launchpad.net/charm-helpers>`_ repository on `Launchpad
<https://launchpad.net/>`_ and syncs it into the charm.  Therefore, **any**
changes done in the ``charmhelpers`` or ``tests/charmhelpers`` directories will
be overwritten during the next sync (which is performed on the charms
automatically at the end of each development cycle).  Note, that although the
project is called "charm-helpers", the directories are named 'charmhelpers'.

Therefore, in order to make changes to the charm-helpers library, this needs to
be done in the separate charm-helpers project and then synced into the charm
using a ``make sync`` command.

From a development work flow perspective, it is *easiest* to first make changes
directly in the ``charmhelpers`` directories in the charm being modified, and
once that is working, then make changes in the charm-helpers project and submit
that as a change.  Then, when the change is committed, it can be synced back
into the charm.

Note, that it is not often that changes are needed in charm-helpers unless they
are core features of all of the related charms.

For details on getting started with `Launchpad`_ development, please read the `Launchpad code page
<https://help.launchpad.net/Code>`_ after you have registered your account.

Also please do reach out to us on IRC or the mailing list. You can find more
details in the :ref:`find-us` page.
