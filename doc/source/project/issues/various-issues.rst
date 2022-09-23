==============
Various issues
==============

This page documents various issues (software limitations/bugs) that may apply
to a Charmed OpenStack cloud. These are still-valid issues that have arisen
during the development cycles of past OpenStack Charms releases. The most
recently discovered issues are documented in the
:doc:`../../release-notes/index` of the latest version of the OpenStack Charms.

.. important::

   It is recommended to read the :doc:`../issues-and-procedures` page before
   continuing.

Lack of FQDN for containers on physical MAAS nodes may affect running services
------------------------------------------------------------------------------

When Juju deploys to a LXD container on a physical MAAS node, the container is
not informed of its FQDN. The services running in the container will therefore
be unable to determine the FQDN on initial deploy and on reboot.

Adverse effects are service dependent. This issue is tracked in bug `LP
#1896630`_ in an OVN and Octavia context. Several workarounds are documented in
the bug.

Adding Glance storage backends
------------------------------

When a storage backend is added to Glance a service restart may be necessary in
order for the new backend to be registered. This issue is tracked in bug `LP
#1914819`_.

.. _ovn_sriov_dhcp:

OVN and SR-IOV: servicing external DHCP and metadata requests
-------------------------------------------------------------

When instances are deployed with SR-IOV networking in an OVN deployment a
change of configuration may be required to retain servicing of DHCP and
metadata requests.

If your deployment has SR-IOV instances, make sure that at least one of the OVN
chassis named applications has the ``prefer-chassis-as-gw`` configuration
option set to 'true'.

The root of the issue is in how Neutron handles scheduling of gateway chassis
for L3 routers and external services differently, and is tracked in bug `LP
#1946456`_.

Ceph RBD Mirror and Ceph Octopus
--------------------------------

Due to an unresolved permission issue the ceph-rbd-mirror charm will stay in a
blocked state after configuring mirroring for pools when connected to a Ceph
Octopus cluster. See bug `LP #1879749`_ for details.

.. LINKS
.. _Release notes: https://docs.openstack.org/charm-guide/latest/release-notes.html

.. BUGS
.. _LP #1896630: https://bugs.launchpad.net/charm-layer-ovn/+bug/1896630
.. _LP #1914819: https://bugs.launchpad.net/charm-glance/+bug/1914819
.. _LP #1946456: https://bugs.launchpad.net/bugs/1946456
.. _LP #1879749: https://bugs.launchpad.net/charm-ceph-rbd-mirror/+bug/1879749
