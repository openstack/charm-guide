===================
Hardware offloading
===================

Overview
--------

As of the 20.05 release, the OpenStack charms support configuration of Open
vSwitch hardware offloading with Mellanox ConnectX-5 NICs. Hardware offloading
can be used to accelerate VLAN and VXLAN networking using the capabilities of
the underlying network card to achieve much higher performance than with virtio
based VM ports.

See the Neutron documentation on `OVS hardware offload`_ for background
information.

For OVN-specific information on hardware offloading see the
:ref:`ovn_configuration` page.

.. warning::

   Hardware offloading cannot be used with either SR-IOV or DPDK networking
   support as provided by the OpenStack charms.

Prerequisites
-------------

* Ubuntu 18.04 LTS or later
* Linux kernel >= 5.3
* Open vSwitch >= 2.11
* OpenStack Stein or later
* Mellanox ConnectX-5 NICs using recent firmware (>= 16.26.4012)

.. note::

   Hardware offload does not currently support offloading of Neutron Security
   Group rules - experimental support is expected in Open vSwitch 2.13 when
   used with Linux >= 5.4 and as yet unreleased NIC firmware. It is recommended
   that port security is disabled on Neutron networks being used for hardware
   offloading use cases due to the performance overhead of enforcing security
   group rules in userspace.

Deployment
----------

MAAS - Hardware Enablement Kernel
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

As a more recent Linux kernel than that provided as part of Ubuntu 18.04 LTS
is required to support this feature, machines with compatible network cards
must be commissioned and configured in MAAS to use the Ubuntu 18.04 LTS
Hardware Enablement (HWE) kernel rather than the standard release kernel.

MAAS - SR-IOV VF-LAG
~~~~~~~~~~~~~~~~~~~~

Mellanox `SR-IOV VF-LAG`_ provides hardware link-aggregation (LAG) for
hardware offloaded ports and is recommended for deployment as it avoids the
need to pass two hardware offloaded ports to each VM for resilience.  This
feature is configured in the underlying NIC using standard Linux bonding as
configured through MAAS.

.. note::

   VF-LAG can only be used with NIC ports that reside on the same underlying
   NIC.

.. note::

   Use of VF-LAG reduces the offloaded port capacity of the card by 50%.

Charm configuration
~~~~~~~~~~~~~~~~~~~

Hardware offload support is enabled using the ``enable-hardware-offload``
option provided by the neutron-api and neutron-openvswitch charms.

Enabling hardware offloading requires configuration of VF representator ports
on the NICs supporting the hardware offload - these are used to route network
packets without flow rules to the OVS userspace daemon for handling and
subsequent programming into the hardware offloaded flows. This is supported
via use of the ``sriov-numvfs`` option provided by the neutron-openvswitch
charm.

Finally the ``openvswitch`` firewall driver must be used with hardware
offloading. Eventually it will be possible to offload security group rules
using this driver (see note above).

The following overlay may be used with the OpenStack base deployment bundle:

.. code-block:: yaml

   series: bionic
   applications:
     neutron-openvswitch:
       charm: cs:neutron-openvswitch
       options:
         enable-hardware-offload: true
         sriov-numvfs: "enp3s0f0:32 enp3s0f1:32"
         firewall-driver: openvswitch
     neutron-api:
       charm: cs:neutron-api
       options:
         enable-hardware-offload: true
     nova-compute:
       options:
         pci-passthrough-whitelist: '{"address": "*:03:*", "physical_network": null}'

In this overlay ``enp3s0f0`` and ``enp3s0f1`` are two ports on the same
Mellanox ConnectX-5 card and are configured as a Linux bond ``bond1`` to enable
VF-LAG for resilience and performance. ``bond1`` is also configured with the
network interface used for VXLAN overlay traffic to allow full offloading of
networks of this type.

The nova-compute charm is configured to use the VF functions provided by the
network cards using the ``pci-passthrough-whitelist`` option. The above example
demonstrates configuration for VXLAN overlay networking.

.. caution::

   After deploying the above example the machines hosting neutron-openvswitch
   units must be rebooted for the changes to take effect.

Creating hardware offloaded ports
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Hardware offloaded ports must be created via Neutron and then passed to Nova
for use by VMs:

.. code-block:: none

   openstack port create --network private --vnic-type=direct \
        --binding-profile '{"capabilities": ["switchdev"]}' direct_port1
   openstack server create --flavor m1.small --image bionic \
        --nic port-id=direct_port1 vm1

The image used for the VM must include the Mellanox kernel driver. Ubuntu 18.04
LTS (or later) cloud images include this driver by default.

.. LINKS
.. _OVS hardware offload: https://docs.openstack.org/neutron/stein/admin/config-ovs-offload.html
.. _SR-IOV VF-LAG: https://docs.mellanox.com/pages/releaseview.action?pageId=25133702
