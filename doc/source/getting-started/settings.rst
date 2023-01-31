======================
Collect local settings
======================

This stage entails the collection of hardware and networking information from
the local environment. In the tables below, the Placeholder column contains
variables (e.g. EXT_GW) that will be referred to (e.g. $EXT_GW) on a later page
where they are to be replaced by the environment's actual values (e.g.
10.246.112.1).

Assign a value to each variable now.

.. list-table::
   :header-rows: 1
   :widths: 16 16 16 40

   * - Hardware attribute
     - Tutorial example value
     - Placeholder
     - Comments

   * - data port interface
     - **bond1**
     - OVN_DATA_PORT
     - the name of the network interface that the three OVN Chassis will bind
       to
   * - storage devices
     - **/dev/sda /dev/sdb /dev/sdc /dev/sdd**
     - OSD_DEVICES
     - all possible block devices that can be used on the three cloud nodes -
       each cloud node will also act as a Ceph OSD node

   * - constraints
     - **cores=4,mem=16**
     - CONSTRAINTS
     - optional - used to filter on MAAS nodes that align with the stated
       hardware requirements

Notes on the OVN Chassis network interface (OVN_DATA_PORT):

* An Open vSwitch (OVS) bridge is always needed (per node), and its name is
  determined by the ovn-chassis charm's ``ovn-bridge-mappings`` option in the
  bundle ('br-ex'). OVN_DATA_PORT denotes the name of the bridge's underlying
  interface (e.g. 'bond1' or 'enp1s0').

* When a second interface *is not* used on a node, an OVS bridge must be set up
  manually (see the :ref:`MAAS page <cdg:ovs_bridge>` in the Deploy Guide for
  help). In the MAAS web UI, this bridge must be associated with the subnet
  (EXT_SUBNET) and its IP address status should be set to either 'Auto assign'
  or 'Static assign'.

* When a second interface *is* used on a node, the charms will automatically
  create the OVS bridge. In the MAAS web UI, this interface must be associated
  with the subnet (EXT_SUBNET), but its IP address status should remain as
  'Unconfigured'.

* It is convenient for the bridges' underlying interface names to be common.
  However, if this is not the case, individual hardware (MAC) addresses can be
  used instead. Make note of them.

.. list-table::
   :header-rows: 1
   :widths: 15 12 15 30

   * - External network attribute
     - Tutorial example value
     - Placeholder
     - Comments

   * - subnet
     - **10.246.112.0/21**
     - EXT_SUBNET
     - a subnet within MAAS

   * - gateway
     - **10.246.112.1**
     - EXT_GW
     - check MAAS settings

   * - nameserver/DNS
     - **10.246.112.3**
     - EXT_DNS
     - check MAAS settings - if undefined, use the MAAS server address

   * - floating IP range start
     - **10.246.116.23**
     - EXT_POOL_START
     - resides within the chosen subnet

   * - floating IP range end
     - **10.246.116.87**
     - EXT_POOL_END
     - resides within the chosen subnet

The MAAS subnet (EXT_SUBNET) need not be dedicated to the cloud. The deployment
simply makes address allocation requests to this address space for the cloud
nodes (including LXD containers). The cloud will need a minimum of 22 addresses
for its nodes plus any that may be used as floating IPs (for SSH connections).

.. important::

   Since OpenStack manages floating IPs they must not be used by MAAS for any
   other purpose. To assure this, configure a "reserved IP range" in MAAS.
   See `How to manage IP ranges`_ in the MAAS documentation. In this context,
   the subnet (EXT_SUBNET) is considered "managed" in MAAS.

When you're done, move on to the :doc:`overlay` page.

.. LINKS
.. _How to manage IP ranges: https://maas.io/docs/how-to-enable-dhcp#heading--how-to-manage-ip-ranges
