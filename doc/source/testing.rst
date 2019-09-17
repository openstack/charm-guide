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

Execute the synthetic code checks (unit tests) for a charm using the tox
environment specific for the version of Python you wish to test.

.. code:: bash

    tox -e py3

.. note:: The environment name specified in the above example is a moving
   target, and you may need to adapt it depending on what is the current
   version of Python used in the test gate.

Unit tests are stored in the ``unit_tests`` folder; when adding features or
changing existing code, please ensure that appropriate unit tests are added
or updated to cover the changes you are making.

Unit tests are written in Python using standard mocking techniques to isolate
the unit tests from the underlying host operating system.

Writing Unit Tests
~~~~~~~~~~~~~~~~~~

Writing software, and functions, is an art.  It's a balance of conciseness,
simplicity, ergonomics and readability.  To produce software that *only* meets
the feature needs, whilst being maintainable and correct for the life of the
program.

Writing unit tests is, similarly, an art.  The objective is to write tests that
correctly determine that what a function *does* is correct, rather than *how*
the function achieves it - with the exception, in some cases, of performance.
But that's another whole ball-game.

The goal is, for each function under test, is to verify that the *outputs* of
the function are correct for comprehensive sets of *inputs* to the function.
What is not the objective is to test the *internal* implementation of the
function.

It's worth exploring what are the *inputs* and *outputs* of a function, and
that depends on whether the function is *pure* or *impure*.

A *pure* function is one that is always returns the same results for the same
set of values passed to the function.  This means that there are no (input)
side-effects or dependencies on any other state outside of the function.
A pure function is analogous, algorithimically, to a mathematical function.
*Pure* functions also can only call other pure functions.  i.e. a pure function
isn't pure if *it* calls a function that is impure.  Impurity at a particular
level 'infects' every caller of that function.

An *impure* function is basically a function that is not a pure function. i.e.
it may depend on a global variable, or obtain inputs from side-effects (such as
reading IO functions).  It is also impure if it has any output side-effects.

So, in addition to the parameters passed to a function, other inputs (within
a function) are accessing functions that access global state.  e.g. the
``config()`` function, relation functions, and reading from files, or the
network.

Also, in addition to the return value of a function, other outputs (within the
function) are writing to files, using the network, calling subprocess calls,
and other IO operations.

So the goal with unit-testing a function depends on the purity of the function:

* Pure functions require no mocking.  The object is to verify that for
  combinations of input parameter values, that the correct return values are
  presented.  As pure functions are pure 'all the way down', no mocking is
  required, as they will always be consistent for any set of inputs.  Pure
  functions are also fantastic opportunities to use property-based testing.
  However, most of the work in a charm is *all about side-effects*, so most
  functions are *impure*.

* Impure functions require mocking.  The goal is to isolate the
  function-under-test from it's side-effects so that *only* the function is
  being checked.  If the function calls *other* impure functions, they should
  be mocked out.  Tests for a function should *only* test that function, and
  not other functions as a by-product.

Strategies for writing Unit Tests
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

It's important to test *what* a function does, not *how* the function does it.
This is basically a re-statement of the idea to not test the implementation of
a function, but rather the output (and with respect to impure functions) the
side-effects.

If the unit test is dependent on the implementation of the function (not with
respect to side-effects), then that *locks in* the implementation, which makes
refactoring much harder.  i.e. To refactor a function that does the *same*
thing, the unit test would also have to change.  This is the sign of a poor
unit test.

Individual unit tests should be small and test *one* activation of the
function-under-test.  This way, behaviour changes during refactoring, or adding
features, will break the smallest number of tests, and show what behaviour has
changed quickly.  Complex unit tests are more fragile and tend to therefore
come with a higher maintenance burden.

With impure functions, it's important to mock out the side-effects so that the
test doesn't also test other side-effect functions.

Functional Testing
==================

.. note:: Most of the OpenStack Charms have been converted to use the Zaza_
   (Py3) functional test framework, though some may still use the legacy
   Amulet_ (Py2) framework.  Naturally, with the sunset of Py2 overall, newly
   authored tests should be in the Zaza_ framework.

Zaza - Functional Tests
~~~~~~~~~~~~~~~~~~~~~~~

Functional tests for a charm are written using the Zaza_ test framework and
should exercise the target charm with a subset of a full OpenStack deployment
to ensure that the charm is able to correctly deploy and configure the
service that is encapsulates.

Typically each charm will test the OpenStack and Ubuntu release combinations
currently supported by Ubuntu.

The OpenStack Charms tests in their current form may be specific to
execution within a tenant on an OpenStack cloud, via the Juju OpenStack
provider, and that is how the third-party-CI executes them.  Future functional
test enhancements may include the ability run the tests against the Juju
OpenStack provider (a cloud) or the Juju LXD provider (all on one machine).

:Full Tests: Executes all Zaza_ gate tests (may take several hours).  The
    full test set does not run automatically on each proposed change.
    After the lower-cost lint, unit, charm-single and smoke tests have
    completed, reviewers can conduct code reviews then optionally trigger the
    full set of Zaza_ tests (see Rechecking).

    To manually execute of all Zaza_ tests on your locally-defined cloud:

.. code:: bash

    tox -e func

:Smoke Tests: Executes a subset (generaly one) of the Zaza_ deployment test
    sets. The smoke test set runs automatically on every proposed patchset.

    To manually execute the Zaza smoke test on your locally-defined cloud:

.. code:: bash

    tox -e func-smoke

:No-Op: Builds a Python virtualenv per definitions in ``tox.ini``,
    which can be useful in test authoring.

    To manually trigger a build of the virtualenv on your local machine, but
    execute no tests:

.. code:: bash

    tox -e func-noop

Test methods are called in lexical sort order, as with most test runners.
However, each individual test method should be idempotent and expected
to pass regardless of run order or Ubuntu:OpenStack combo.  When writing
or modifying tests, ensure that every individual test is not dependent
on another test method.

Some tests may need to download files from the Internet, such as glance
images. If a web proxy server is required in the environment, the
``OS_HTTP_PROXY`` environment variable must be set. This is unrelated
to Juju's http-proxy settings.

See ``tox.ini`` to determine specifically which test targets will be executed by
each tox target.  Zaza_ test calls are defined in the ``tests/`` directory for
classic charms, and in the ``src/tests/`` directory for layered source charms.


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
.. _Zaza: https://zaza.readthedocs.io/en/latest/
