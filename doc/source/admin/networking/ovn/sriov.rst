=====================
Using SR-IOV with OVN
=====================

.. important::

   This page has been identified as being affected by the breaking changes
   introduced between versions 2.9.x and 3.x of the Juju client. Read
   support note :ref:`juju_29_3x_changes` before continuing.

Single root I/O virtualisation (SR-IOV) is an open hardware specification that
enables splitting a single physical network port into multiple virtual network
ports known as virtual functions (VFs). In this context, the physical port
becomes known as a physical function (PF). OVN allows for the integration of
SR-IOV into OpenStack.

.. note::

   For general information on OVN, refer to the main :doc:`index` page.

A VF can then be directly attached to a VM and thereby bypass the networking
stack of the hosting hypervisor. The main use case is to support applications
with high bandwidth requirements. For such applications, the normal plumbing
through the userspace virtio driver in QEMU will consume too much resources
from the host. For an upstream OpenStack overview see `SR-IOV`_ in the Neutron
documentation.

.. caution::

   Enabling SR-IOV will necessitate the reboot of the associated physical host.

Requirements
------------

SR-IOV support must be present in:

* the PCIe hardware device
* the host operating system
* the host BIOS (and be enabled)

An IOMMU driver and any other pertinent features available to the host's CPU
must be enabled in the kernel manually (e.g. for Intel: ``intel_iommu=on``,
``iommu=pt``, ``probe_vf=0``) or through a bare metal provisioning layer (for
example `MAAS`_).

As per a normal Charmed OpenStack cloud, a mapping between physical network
name, physical port, and OVS bridge should exist. This document presumes the
following options are set with certain values (adjust any further instructions
as per your environment):

* ``ovn-bridge-mappings=physnet1:br-ex``
* ``bridge-interface-mappings=br-ex:enp3s0f0``

OpenStack configuration
-----------------------

There are a number of stages to OpenStack configuration for SR-IOV to become
usable.

Enable SR-IOV
~~~~~~~~~~~~~

To enable base SR-IOV support:

.. code-block:: none

   juju config neutron-api enable-sriov=true
   juju config ovn-chassis enable-sriov=true
   juju integrate ovn-chassis:amqp rabbitmq-server:amqp

Create VFs
~~~~~~~~~~

Map the existing physical network to the physical port and create some VFs on
it (four here):

.. code-block:: none

   juju config ovn-chassis sriov-device-mappings=physnet1:enp3s0f0
   juju config ovn-chassis sriov-numvfs=enp3s0f0:4

.. note::

   The total number of VFs supported by a device can be obtained from the
   device documentation. Post SR-IOV enablement, device files on the host can
   be inspected. For example:

   .. code-block:: none

      cat /sys/class/net/enp3s0f0/device/sriov_totalvfs
      63

   Once SR-IOV is enabled, in the advent that option ``srivo-numvfs`` is
   modified, you can have Netplan attempt to make the necessary changes with
   command :command:`sudo netplan apply`. However, rebooting the underlying
   host remains the best method since changing SR-IOV configuration via Netplan
   is device/driver/configuration specific. 

Reboot physical hosts
^^^^^^^^^^^^^^^^^^^^^

After analysing your cloud's topology and ascertaining what effects a reboot
may have, plan to have each each hypervisor that is hosting an affected
ovn-chassis unit rebooted.

Authorise passthrough devices
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

First gather some device information by querying the ovn-chassis application:

.. code-block:: none

   juju exec -a ovn-chassis 'lspci -nn | grep "Virtual Function"'

   03:10.0 Ethernet controller [0200]: Intel Corporation 82599 Ethernet Controller Virtual Function [8086:10ed] (rev 01)
   03:10.2 Ethernet controller [0200]: Intel Corporation 82599 Ethernet Controller Virtual Function [8086:10ed] (rev 01)
   03:10.4 Ethernet controller [0200]: Intel Corporation 82599 Ethernet Controller Virtual Function [8086:10ed] (rev 01)
   03:10.6 Ethernet controller [0200]: Intel Corporation 82599 Ethernet Controller Virtual Function [8086:10ed] (rev 01)

Here, the physical device vendor_id and product_id are given, respectively, by
``[8086:10ed]``.

Using this information (and the known physical network name), inform the
Compute service of the PCI device (PF) it can pass through to VMs:

.. code-block:: none

   juju config nova-compute pci-passthrough-whitelist=\
      '{"vendor_id":"8086", "product_id":"10ed", "physical_network":"physnet1"}'

Create a Neutron direct port
----------------------------

Create an SR-IOV port (type ``direct``) in Neutron (as opposed to using a
traditional port (type ``virtio``). Here it is created on network 'ext_net' and
named after our intended VM name (jammy-3) as each VM will require its own
port:

.. code-block:: none

   openstack port create --network ext_net --vnic-type direct sriov-jammy-3

Configure for DHCP
------------------

In an SR-IOV/OVN context, Neutron and the charms take care of making DHCP
available automatically. The operator should however ensure that:

#. there is a mapping between physical network name, physical port, and OVS
   bridge as described in the `Requirements`_ section (this allows OVN to
   configure an 'external' port on one of the Chassis for providing DHCP and
   metadata to instances connected directly to the network through SR-IOV).
#. DHCP is enabled in Neutron on the subnet associated with the network on
   which the direct port was created (i.e. :command:`openstack port create`
   above)
#. one of the involved ovn-chassis applications has option
   ``prefer-chassis-as-gw`` set to 'true' (see issue :ref:`ovn_sriov_dhcp` for
   the reasoning)

See the Neutron `SR-IOV guide for OVN`_ for more information.

Create a VM
-----------

Create a VM and attach it to the SR-IOV port:

.. code-block:: none

   openstack server create \
      --image jammy-amd64 --flavor m1.micro --key-name admin-key \
      --network int_net --nic port-id=sriov-jammy-3 \
      jammy-3

Inspect the VM's assigned interface
-----------------------------------

Query the VM (here 203.0.113.1) for the assigned VF (via the PF):

.. code-block:: none

   ssh -i ~/cloud-keys/admin-key ubuntu@203.0.113.1 | lspci -vnn | grep -A9 '\[8086:10ed\]'

   00:05.0 Ethernet controller [0200]: Intel Corporation 82599 Ethernet Controller Virtual Function [8086:10ed] (rev 01)
           Subsystem: Intel Corporation 82599 Ethernet Controller Virtual Function [8086:7b11]
           Physical Slot: 5
           Flags: bus master, fast devsel, latency 0
           Memory at febf0000 (64-bit, prefetchable) [size=16K]
           Memory at febf4000 (64-bit, prefetchable) [size=16K]
           Capabilities: <access denied>
           Kernel driver in use: ixgbevf
           Kernel modules: ixgbevf

Here ``ixgbevf``, the Linux VF driver for Intel is in use.

.. LINKS
.. _MAAS: https://maas.io/docs/how-to-customise-machines#heading--how-to-set-global-kernel-boot-options
.. _SR-IOV: https://docs.openstack.org/neutron/latest/admin/config-sriov.html
.. _SR-IOV guide for OVN: https://docs.openstack.org/neutron/latest/admin/ovn/sriov.html
.. _LP #1946456: https://bugs.launchpad.net/bugs/1946456
