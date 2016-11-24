.. _testing:

=======
Testing
=======

Every proposed change to a charm is run through testing during the review
verification process.  If you want to contribute a change or fix to a charm,
please take time to review the `Unit Testing`_ and `Functional Testing`_
sections of this document.

OpenStack Charm CI will verify your changes, but please execute at least
unit tests locally before submitting patches to reduce load on the OpenStack
CI infrastructure.

The OpenStack Charms are compliant with the OpenStack
`Consistent Testing Interface <https://governance.openstack.org/reference/cti/python_cti.html>`__;
take a read on how this works to understand in full.

Lint
====

You can verify the compliance of your code changes using flake8 and the charm
proof tool using the pep8 tox environment:

.. code:: bash

    tox -e pep8

Ensure that any non-compliance is corrected prior to raising/updating a review.

Unit Testing
============

Execute the synthetic code checks (unit tests) for a charm using the tox py27
environment:

.. code:: bash

    tox -e py27

Unit tests are stored in the ``unit_tests`` folder; when adding features or
changing existing code, please ensure that appropriate unit tests are added
or updated to cover the changes you are making.

Unit tests are written in Python using standard mocking techniques to isolate
the unit tests from the underlying host operating system.

Functional Testing
==================

Amulet
~~~~~~

Functional tests for a charm are written using the Amulet_ test framework and
should exercise the target charm with a subset of a full OpenStack deployment
to ensure that the charm is able to correctly deploy and configure the
service that is encapsulates.

The OpenStack charm helpers provide some Amulet deployment helpers to ease
testing of different OpenStack release combinations; typically each charm will
test the OpenStack and Ubuntu release combinations currently supported by
Ubuntu.

The OpenStack Charms Amulet tests in their current form may be specific to
execution within a tenant on an OpenStack cloud, via the Juju OpenStack
provider, and that is how the third-party-CI executes them.  Future functional
test enhancements include the ability run the tests against the Juju OpenStack
provider (a cloud) or the Juju LXD provider (all on one machine).

:Full Amulet: Executes all Amulet gate tests (may take several hours).  The
    full Amulet test set does not run automatically on each proposed change.
    After the lower-cost lint, unit, charm-single and Amulet-smoke tests have
    completed, reviewers can conduct code reviews then optionally trigger the
    full set of Amulet tests (see Rechecking).

    To manually trigger execution of all Amulet tests on your locally-defined
    cloud:

.. code:: bash

    tox -e func27

:Amulet Smoke: Executes a subset (generaly one) of the Amulet deployment test
    sets. The Amulet smoke test set does run automatically on every proposed
    patchset.
    
    To manually trigger execution of the Amulet smoke test on your
    locally-defined cloud:

.. code:: bash

    tox -e func27-smoke

:No-Op: Builds a Python virtualenv per definitions in ``tox.ini``,
    which can be useful in test authoring.

    To manually trigger a build of the virtualenv on your local machine, but
    execute no tests:

.. code:: bash

    tox -e func27-noop

Test methods are called in lexical sort order, as with most test runners.
However, each individual test method should be idempotent and expected
to pass regardless of run order or Ubuntu:OpenStack combo.  When writing
or modifying tests, ensure that every individual test is not dependent
on another test method.

Some tests may need to download files from the Internet, such as glance
images. If a web proxy server is required in the environment, the
``AMULET_HTTP_PROXY`` environment variable must be set. This is unrelated
to Juju's http-proxy settings.

See ``tox.ini`` to determine specifically which test targets will be executed by
each tox target.  Amulet tests reside in the ``tests/`` directory for classic
charms, and in the ``src/tests/`` directory for layered source charms.


Rechecking
==========

*BEFORE issuing a recheck of any kind, please inspect the CI results and
log artifacts to understand the failure reason.*

*Rechecks should only be used in the event of a system failure (not for
race conditions or problems introduced by the proposed code changes).*

*Developers are expected to have executed tests prior to submitting patches.*

Tests can be retriggered, or additional tests can be requested, simply by
replying on the Gerrit review with one of the recognized magic phrases below.

``recheck``
    Re-triggers events as if a new patchset had been submitted, including
    all defined OpenStack Infra tests AND third-party-CI tests.

``charm-recheck``
    Re-triggers only the default set of OpenStack Charms third-party-ci tests,
    but not the OpenStack Infra tests.  *Depending on system load and which
    charm is under test, this will typically take 30 to 60 minutes.*

``charm-recheck-full``
    Triggers a full set of OpenStack Charms third-party-ci tests, but not the
    OpenStack Infra tests.  *This will take several hours.*

.. _Amulet: https://jujucharms.com/docs/devel/tools-amulet
