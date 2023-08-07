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

Neutron ML2 OVS plugin on DVR mode
----------------------------------

Environments configured to use the ML2 OVS plugin in DVR mode and have
configured an external network of type ``flat`` will be affected by bug `LP
#2015090`_. The symptom of an affected system is that newly launched instances
won't have access to the Metadata service and the ``neutron-dhcp-agent``
service log will contain the following error:

.. code-block:: none

   [...]
   2023-03-31 19:35:06.095 58625 ERROR neutron.agent.dhcp.agent return self._name[:constants.DEVICE_NAME_MAX_LEN]
   2023-03-31 19:35:06.095 58625 ERROR neutron.agent.dhcp.agent TypeError: 'bool' object is not subscriptable

glance-simplestreams-sync: endpoint change
------------------------------------------

The ceph-radosgw charm improves how URLs are processed by the RADOS Gateway.
This change however will lead to breakage for an existing ``product-streams``
endpoint, set up by the glance-simplestreams-sync application. Manual
intervention is required - see the :ref:`Upgrade issues
<charm_upgrade_issue-radosgw_gss>` page for more information.

Nova fails to parse new libvirt mediated device name format
-----------------------------------------------------------

The name format of mediated devices in libvirt was recently changed from
``mdev_<uuid>`` to ``mdev_<uuid>_<parent>``. For users of the new tech-preview
nova-compute-nvidia-vgpu charm, this will cause new Nova instances to enter
into an error state subsequent to a vGPU device being attached to an instance.
This is being tracked in issue `LP #1951656`_. A fix will soon be available as
an SRU.

glance-simplestreams-sync: Juju resources support and snap install
------------------------------------------------------------------

The glance-simplestreams-sync charm has gained support for Juju resources.
Since Simplestreams is now installed via a snap, offline environments can
benefit by being able to supply the snap as a resource.

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
.. _LP #1951656: https://bugs.launchpad.net/nova/+bug/1951656
.. _LP #2015090: https://bugs.launchpad.net/ubuntu/+source/neutron/+bug/2015090
