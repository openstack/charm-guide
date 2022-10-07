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
     - **/dev/sda /dev/sdb**
     - OSD_DEVICES
     - all possible block devices that can be used on the three cloud nodes -
       each cloud node will also act as a Ceph OSD node

   * - constraints
     - **cores=4,mem=16**
     - CONSTRAINTS
     - optional - used to filter on MAAS nodes that align with the stated
       hardware requirements

Notes on the OVN Chassis network interface (OVN_DATA_PORT) when a second
interface is used (i.e. Open vSwitch bridge is not set up within MAAS):

* As given in the table, it is convenient for the interface's name to be common
  across all three cloud nodes. If this is not the case, individual hardware
  (MAC) addresses can be used instead. Make note of them.

* The interfaces must be cabled on the MAAS subnet. Within the MAAS web UI,
  the interfaces' subnet name and their IP addresses' status must both remain
  as 'Unconfigured'.

* The ovn-chassis charm will dynamically create an OVS bridge on each
  interface. An OVS bridge is always required on each OVN Chassis node.

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
     - **10.246.116.0**
     - EXT_POOL_START
     - resides within the chosen subnet

   * - floating IP range end
     - **10.246.116.10**
     - EXT_POOL_END
     - resides within the chosen subnet

The MAAS subnet (EXT_SUBNET) need not be dedicated to the cloud. The deployment
simply makes address allocation requests to this address space for the cloud
nodes (including LXD containers). The cloud will need a minimum of 22 addresses
for its nodes plus any that may be used as floating IPs (for SSH connections).

.. important::

   Since OpenStack manages floating IPs they must not be used by MAAS for any
   other purpose. This can be assured by means of a `Reserved IP range`_.

When you're done, move on to the :doc:`Configure the overlay bundle <overlay>`
page.

.. LINKS
.. _Reserved IP range: https://maas.io/docs/maas-concepts-and-terms-reference#heading--ip-ranges
