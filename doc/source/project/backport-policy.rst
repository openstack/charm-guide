==================
Backporting policy
==================

Once a release of Charmed OpenStack has been published, it is subject to wider
use. In particular it is important to note that stable releases are run in
production environments, with an expectation that the release maintains
stability. While users expect stability, they also expect bug fixes. The
balance of providing these fixes while maintaining stability is extremely
important.

Process
-------

Anyone can propose that a fix be backported to a stable branch.

Requirements
~~~~~~~~~~~~

The below requirements must be met for a backport to be accepted.

* **reverse chronological order** - in order to be accepted into a given stable
  branch, it must be merged into all newer stable branches first, including the
  development branch

* **cherry pick line** - it must include a "cherry picked from commit ..." line
  in its commit message

* **CI testing** - if it relies on functional testing, the tests must be
  backported in tandem (e.g. `Refactor unit tests to avoid leaks of mocks`_)

Backport candidates
-------------------

Only a fix for a bug classified as **Critical** or **High** in Launchpad can be
considered as a backport candidate.

Exclusions
~~~~~~~~~~

A code change is generally not eligible as a backport when it is either:

* large
* risky

The following consequences of a code change qualifies it as risky:

* new behaviour is introduced in the charm
* new behaviour is introduced in the payload
* new Debian packages are installed on the host
* new Python packages are installed in the charm

The above is not an exhaustive list.

Common backport request types
-----------------------------

The following sections cover common types of backport requests.

Charm configuration options
~~~~~~~~~~~~~~~~~~~~~~~~~~~

A backport of a simple charm configuration enablement that is opt-in is a
candidate to be backported.

For changes to configuration options, new behaviour should not be introduced.
This can take the form of default values changed in comparison to the ones
already defined in the configuration template or the payload's default.

Here is an example of a patch that introduced a new configuration option
without changing the default behaviour: `Support --visibility option for
simplestreams`_

Juju actions
~~~~~~~~~~~~

A Juju action that improves operator experience is a candidate to be
backported.

For changes to actions, the stable maintainers team takes extra care to asses
what risk the new action introduces. If you consider the risk to be minimal,
then include your reasoning, so the maintainers team can take a more informed
decision.

In general, actions that execute only read operations on the system are
considered low risk.

Backport approvals
------------------

Charms
~~~~~~

All charm stable branch reviews must be approved by **two** core reviewers or
stable maintainers prior to landing and CI system(s) voted with a Verified+1.

Charm and test libraries
~~~~~~~~~~~~~~~~~~~~~~~~

Backports are also sometimes needed for the charm libraries that support the
OpenStack, Ceph, OVN and support charms.  These libraries are:

* `zaza`_
* `zaza-openstack-tests`_
* `charm-helpers`_
* `charms.openstack`_
* `charms.ceph`_

All library backports may be approved by a **single** core reviewer or stable
maintainer prior to landing and the CI system(s) having voted with a
Verified+1.

.. note::

   This is different to charm backports as libraries are "tested again" when
   charms are built and go through the test process.

Cherry picking details
----------------------

Backports are done by cherry picking a patch that has been already merged into
a more recent branch (e.g. ``master``).

The simplest way to do this is with the `"Cherry Pick" button`_ in the web UI
for the original patch (e.g. ``master``). Gerrit will automatically create a
new review and include the cherry pick line in its commit message.

However, you can also use `git-review`_ to manually propose a backport. For
example, to propose a backport from the ``master`` branch to the ``zed``
branch:

.. code-block:: none

   git checkout -t origin/stable/zed
   git cherry-pick -x <master_commit_id>
   git review stable/zed

The ``-x`` option in the :command:`cherry-pick` command inserts the cherry pick
line in the commit message.

.. LINKS
.. _"Cherry Pick" button: https://gerrit-review.googlesource.com/Documentation/user-review-ui.html#cherry-pick
.. _git-review: https://docs.opendev.org/opendev/git-review/latest/
.. _Support --visibility option for simplestreams: https://review.opendev.org/q/I1955f3d2a56654c9a683a2b9d36b33c0f0fd63d4
.. _Refactor unit tests to avoid leaks of mocks: https://review.opendev.org/c/openstack/charm-nova-compute/+/874505
.. _zaza: https://github.com/openstack-charmers/zaza
.. _zaza-openstack-tests: https://github.com/openstack-charmers/zaza-openstack-tests
.. _charm-helpers: https://github.com/juju/charm-helpers
.. _charms.openstack: https://opendev.org/openstack/charms.openstack
.. _charms.ceph: https://opendev.org/openstack/charms.ceph
