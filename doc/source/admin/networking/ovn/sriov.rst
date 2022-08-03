=====================
Using SR-IOV with OVN
=====================

Single root I/O virtualisation (SR-IOV) enables splitting a single physical
network port into multiple virtual network ports known as virtual functions
(VFs). The division is done at the PCI level which allows attaching the VF
directly to a virtual machine instance, bypassing the networking stack of the
hypervisor hosting the instance.

.. note::

   For general information on OVN, refer to the main :doc:`index` page.

The main use case for this feature is to support applications with high
bandwidth requirements. For such applications the normal plumbing through the
userspace virtio driver in QEMU will consume too much resources from the host.

It is possible to configure chassis to prepare network interface cards (NICs)
for use with SR-IOV and make them available to OpenStack.

Prerequisites
-------------

To use the feature you need to use a NIC with support for SR-IOV.

Machines need to be pre-configured with appropriate kernel command-line
parameters. The charms do not handle this facet of configuration and it is
expected that the user configure this either manually or through the bare metal
provisioning layer (for example `MAAS`_). Example:

.. code-block:: none

   intel_iommu=on iommu=pt probe_vf=0

Charm configuration
-------------------

Enable SR-IOV, map physical network name 'physnet2' to the physical port named
'enp3s0f0' and create four virtual functions on it:

.. code-block:: none

   juju config neutron-api enable-sriov=true
   juju config ovn-chassis enable-sriov=true
   juju config ovn-chassis sriov-device-mappings=physnet2:enp3s0f0
   juju config ovn-chassis sriov-numvfs=enp3s0f0:4

.. caution::

   After deploying the above example the machines hosting ovn-chassis
   units must be rebooted for the changes to take effect.

After enabling the virtual functions you should take note of the ``vendor_id``
and ``product_id`` of the virtual functions:

.. code-block:: none

   juju run --application ovn-chassis 'lspci -nn | grep "Virtual Function"'

.. code-block:: console

   03:10.0 Ethernet controller [0200]: Intel Corporation 82599 Ethernet Controller Virtual Function [8086:10ed] (rev 01)
   03:10.2 Ethernet controller [0200]: Intel Corporation 82599 Ethernet Controller Virtual Function [8086:10ed] (rev 01)
   03:10.4 Ethernet controller [0200]: Intel Corporation 82599 Ethernet Controller Virtual Function [8086:10ed] (rev 01)
   03:10.6 Ethernet controller [0200]: Intel Corporation 82599 Ethernet Controller Virtual Function [8086:10ed] (rev 01)

In the above example ``vendor_id`` is '8086' and ``product_id`` is '10ed'.

Add a mapping between physical network name, physical port, and Open vSwitch
bridge:

.. code-block:: none

   juju config ovn-chassis ovn-bridge-mappings=physnet2:br-ex
   juju config ovn-chassis bridge-interface-mappings br-ex:a0:36:9f:dd:37:a8

.. note::

   The above configuration allows OVN to configure an 'external' port on one
   of the Chassis for providing DHCP and metadata to instances connected
   directly to the network through SR-IOV.

For OpenStack to make use of the VFs, the ``neutron-sriov-agent`` needs to talk
to RabbitMQ:

.. code-block:: none

   juju add-relation ovn-chassis:amqp rabbitmq-server:amqp

OpenStack Nova also needs to know which PCI devices it is allowed to pass
through to instances:

.. code-block:: none

   juju config nova-compute pci-passthrough-whitelist='{"vendor_id":"8086", "product_id":"10ed", "physical_network":"physnet2"}'

Boot an instance
----------------

OpenStack can now be directed to boot an instance and attach it to an SR-IOV
port.

First create a port with ``vnic-type`` 'direct':

.. code-block:: none

   openstack port create --network my-network --vnic-type direct my-port

Then create an instance connected to the newly created port:

.. code-block:: none

   openstack server create --flavor my-flavor --key-name my-key \
      --nic port-id=my-port my-instance

.. LINKS
.. _MAAS: https://maas.io/
