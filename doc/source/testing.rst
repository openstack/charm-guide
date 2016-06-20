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

Execute the unit tests for a charm using the tox py27 environment:

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

Execute of Amulet tests currently requires use of the Makefile:

.. code:: bash

    make functional_test

This will execute the full suite of Amulet tests under the ``tests/`` folder.

The ``tests/README`` file in each charm will contain more details on how to
name tests and how to execute them individually.

.. _Amulet: https://jujucharms.com/docs/devel/tools-amulet
