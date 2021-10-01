.. _backporting:

Backporting Policy
==================

This page documents the OpenStack Charms backport policy.

Backport candidates
-------------------

-  Critical and High bugs fixes, if reported in Launchpad.

Backport exclusions
-------------------

-  Medium and Low bug fixes
-  Features

Backport exceptions
-------------------

Exceptions may be made to changes covered under 'Backport exclusions'
in the event that the default behaviour of the charm is not impacted
and that the risk of regression is deemed sufficiently low.

The change must be covered by a bug report and the regression risk
must be evaluated and documented in the bug report as part of the
stable backport process.

Backport approvals
------------------

All charm stable branch reviews must be approved by two core reviewers
prior to landing.
