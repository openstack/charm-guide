Making a change
===============

Development Workflow
~~~~~~~~~~~~~~~~~~~~

Broadly the workflow for making a change to a charm is:

.. code:: bash

    git clone http://github.com/openstack/charm-cinder
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
mark them fix commited when changes are merged.

Execute pep8 and unit tests:

.. code:: bash

    tox

Finally, submit your change for review (if they pass pep8 and unit tests!):

.. code:: bash

    git review

This will push your proposed changes to Gerrit and provide you with a URL for the
review board on http://review.openstack.org/.

To make amendments to your proposed change, update your local branch and then:

.. code:: bash

    git commit --amend
    git review

Stable charm updates
~~~~~~~~~~~~~~~~~~~~

Any update to a stable charm must first be applied into the master branch; it should
then be cherry-picked in a review for the stable branch corresponding to your target
release (ensuring that any interim releases have the fix landed):

.. code:: bash

    git checkout -b stable/bug/XXXX stable/YYYY
    git cherry-pick -x <hash of master branch commit>
    git review

Where XXXX is the launchpad bug ID corresponding to the fix you want to backport and
YYYY is the release name you are targeting e.g. 16.04

.. note:: when cherry-picking a commit and/or modifying the commit message, always ensure that
          the original Change-Id is left intact.
