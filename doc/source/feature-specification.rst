.. _feature_specification:

=====================
Feature Specification
=====================

The OpenStack Charms project uses specifications to identify and define the
implementation details of new major features. Features that require new charm
configuration options, new charm interfaces, and new charm relations should
have an agreed spec prior to proposing code for review. Likewise, a spec
should be landed before work begins on wholly new charms.

The following resources provide reference to existing accepted specs, the
source code repository, recent activity on that repository, and a template
to use as a base for new specs.

* `published charm specs`_
* `charm-specs source repository`_
* `charm-specs history`_
* `charm-specs template`_

Recommended work flow for introducing a new feature specification:

1. Have a :doc:`conversation <find-us>` with core charmers to avoid overlapping
   or duplicate work.

2. Discuss the high-level design with core charmers to arrive at an optimal
   approach (before writing code).

3. Compose the specification with the aim to remove all doubt as to "what it is"
   and "what it isn't."  This should be based on the `charm-specs template`_
   and posted as a Gerrit review.

4. Include one or more specific user stories or use cases which support the
   practical application of the feature specification.

5. Support the proposal by including relevant bug references in the
   specification.

6. Circulate the proposal on the :doc:`mailing list <find-us>` to ensure wider
   visibility.

7. Engage in :doc:`further discussion <find-us>` surrounding the spec proposal
   and the mailing list post so that it can be reviewed and landed.

.. LINKS
.. _published charm specs: https://specs.openstack.org/openstack/charm-specs/
.. _charm-specs source repository: https://opendev.org/openstack/charm-specs
.. _charm-specs history: https://review.opendev.org/q/project:openstack/charm-specs+status:merged
.. _charm-specs template: https://opendev.org/openstack/charm-specs/src/branch/master/specs/template.rst
