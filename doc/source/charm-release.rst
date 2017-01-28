.. _charm-release:

Release Policy
==============

The OpenStack charm repositories are all tagged as `release:independent`; however the
OpenStack Charms project does have a regular release cadence which makes specific demands
on the charms in terms of development and testing.

The OpenStack Charms team produces a release every 6 months aligned to the main
OpenStack release cadence, and an interim release 3 months after each main release.

To be included as part of a release, the (sub)team supporting a charm must meet the
following release requirements:

1. Support for the latest OpenStack release, if the charm release is aligned to the
   main OpenStack release cadence; this means that supporting projects need to produce
   supporting releases alongside the main OpenStack release.

2. Charms must include functional tests to be eligible for inclusion in the official
   OpenStack Charm release every 3 months; these are typically implemented as
   Amulet tests (https://jujucharms.com/docs/stable/tools-amulet) within each charm.

3. Charms must have an active and responsive community of developers to help sustain the
   lifecycle of the charm. Charms that do not have any active community participation will
   be reviewed at a 3 or 6 month cadence.

4. Charms should provide 3rd party CI where it's not possible to provide functional
   testing of charms using standard OpenStack Cloud resources; examples of this might
   include (but are not limited to) SDN and Storage integrations that rely on
   specific vendor hardware or proprietary software.

Charms which don't meet these requirements can continue to be part of the
OpenStack Charms project, but won't form part of the official 3 monthly charm
release.  This approach allows new charms to incubate as part of the wider
OpenStack Charms project, with inclusion in the 3-monthly release when this
policy is met.

Charms may choose to opt-out of the co-ordinated charm release, and follow
a more independent release approach - this may be appropriate for supporting
charms in the wider OpenStack ecosystem which are not aligned to the main
OpenStack release cycle.

This policy is broadly based on the Charm Store 'curated charm' policy adopted
by the wider charm community, and as such charms not meeting the OpenStack
Charms release policy will not form part of the curated charm set on the
Juju Charm Store. For more information on 'curated charm' policy, 
please review https://jujucharms.com/docs/stable/authors-charm-policy.
