==================
Backporting policy
==================

Once a release of Charmed OpenStack has been published, it is subject to wider
use. In particular it is important to note that stable releases are run in
production environments, with an expectation that the release maintains
stability. While users expect stability, they also expect bug fixes. The balance
of providing these fixes while maintaining stability is extremely important.

Process
-------

All backport candidates must be merged into the current development branch.

Backports must be proposed in reverse chronological order. In order to accept a
change into a given stable branch, it must be merged into all newer stable
branches first, including the development branch.

All backports must include the footer "cherry picked from commit ...".

Backport candidates
-------------------

Bug fixes classified as **Critical** or **High** in Launchpad are considered
candidates to be backported.

A backport of a simple configuration enablement that is opt-in is a candidate to
be backported.

A juju action that improves operator experience is a candidate to be backported.

Exclusions
----------

A large or risky new feature is not a candidate for being backported.

An inconclusive list of aspects that make a backport to fall in the category of
"risky new feature" would be:

* It introduces new behaviors to the charm or the payload
* It installs new debian packages in the system, or python packages in the
  charm.


Charm configuration options
---------------------------

It’s important patches don’t introduce new behaviors. This can take the form of
default values changed in comparison to the ones already defined in the
configuration template or the payload's default.

This is an example of a patch that introduced a new configuration option without
changing the default behavior: `Support --visibility option for simplestreams`_

Juju actions
------------

The stable maintainers team takes extra care to asses what risk the new action
introduces, if you consider the risk to be minimal, then include your reasoning,
so the maintainers team can take a more informed decision.

In general actions that execute only read operations on the system are
considered low risk.

CI testing
----------

Commits that depend on functional testing need to be backported as well in
tandem, for example: `Refactor unit tests to avoid leaks of mocks`_.

Backport approvals
------------------

All charm stable branch reviews must be approved by two core reviewers or stable
maintainers prior to landing and CI system(s) voted with a Verified+1.

Proposing Fixes
---------------

Anyone can propose a fix to the stable maintainers team.

The OpenStack Charms project uses git for source version control, hence
backports are done by cherry picking a patch that has been already merged into
another branch (e.g. ``master``).

The simplest way to cherry-pick a change is to use the `"Cherry Pick" button`_
in the web UI for the original patch in ``master``. Gerrit will take care of
creating a new review, modifying the commit message to include "cherry picked
from commit ..."

You can use `git-review`_ to propose a change to the ``zed`` branch with:

.. code::

   $ git checkout -t origin/stable/zed
   $ git cherry-pick -x $master_commit_id
   $ git review stable/zed

.. note::

   cherry-pick -x option includes 'cherry-picked from ...' line in the commit
   message, required by policy.

.. LINKS
.. _"Cherry Pick" button: https://gerrit-review.googlesource.com/Documentation/user-review-ui.html#cherry-pick
.. _git-review: https://docs.opendev.org/opendev/git-review/latest/
.. _Support --visibility option for simplestreams: https://review.opendev.org/q/I1955f3d2a56654c9a683a2b9d36b33c0f0fd63d4
.. _Refactor unit tests to avoid leaks of mocks: https://review.opendev.org/c/openstack/charm-nova-compute/+/874505
