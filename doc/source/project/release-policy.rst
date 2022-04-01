==============
Release policy
==============

The OpenStack charm repositories are all tagged as `release:independent`; however the
OpenStack Charms project does have a regular release cadence which makes specific demands
on the charms in terms of development and testing.

The OpenStack Charms team produces a release every 6 months, with every other release
aligned to the main OpenStack release.

Each stable release of OpenStack Charms is backwards-compatible to cover all currently-supported
combinations of Ubuntu + OpenStack.  It follows that the latest stable charm revision should
be used before proceeding with topological changes, charm application migrations, workload
upgrades, series upgrades, or bug reports.

To be included as part of a release, the (sub)team supporting a charm must meet the
following release requirements:

1. Support for the latest OpenStack release, if the charm release is aligned to the
   main OpenStack release cadence; this means that supporting projects need to produce
   supporting releases alongside the main OpenStack release.

2. Charms must include functional tests to be eligible for inclusion in the official
   OpenStack Charm release every 6 months; these are typically implemented as
   Zaza tests within each charm.

3. Charms must have an active and responsive community of developers.

4. Charms should provide 3rd party CI where it's not possible to provide functional
   testing of charms using standard OpenStack Cloud resources; examples of this might
   include (but are not limited to) SDN and Storage integrations that rely on
   specific vendor hardware or proprietary software.

Charms which don't meet these requirements can continue to be part of the
OpenStack Charms project, but won't form part of the official 6 monthly charm
release.  This approach allows new charms to incubate as part of the wider
OpenStack Charms project, with inclusion in the 6-monthly release when this
policy is met.

Charms may choose to opt-out of the coordinated charm release, and follow
a more independent release approach - this may be appropriate for supporting
charms in the wider OpenStack ecosystem which are not aligned to the main
OpenStack release cycle.

This policy is broadly based on the Charm Store 'curated charm' policy adopted
by the wider charm community, and as such charms not meeting the OpenStack
Charms release policy will not form part of the curated charm set on the
Juju Charm Store.
