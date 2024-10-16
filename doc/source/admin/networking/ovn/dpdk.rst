======================
Enabling DPDK with OVN
======================

The OVN chassis can be configured to use experimental DPDK userspace network
acceleration.

.. note::

   For general information on OVN, refer to the main :doc:`index` page.

.. note::

   Instances are required to be attached to a external network (also known as
   provider network) for connectivity. OVN supports distributed DHCP for
   provider networks. For OpenStack workloads, the use of `Nova config drive`_
   is required to provide metadata to instances.

Prerequisites
-------------

To use the feature you need to use a supported CPU architecture and network
interface card (NIC) hardware. Please consult the `DPDK supported hardware
page`_.

Machines need to be pre-configured with appropriate kernel command-line
parameters. The charm does not handle this facet of configuration and it is
expected that the user configure this either manually or through the bare metal
provisioning layer (for example `MAAS`_).

Example:

.. code-block:: none

   default_hugepagesz=1G hugepagesz=1G hugepages=64 intel_iommu=on iommu=pt

For the communication between the host userspace networking stack and the guest
virtual NIC driver to work the instances need to be configured to use
hugepages. For OpenStack this can be accomplished by `Customising instance huge
pages allocations`_.

Example:

.. code-block:: none

   openstack flavor set m1.large --property hw:mem_page_size=large

By default, the charms will configure Open vSwitch/DPDK to consume one
processor core + 1G of RAM from each NUMA node on the unit being deployed. This
can be tuned using the ``dpdk-socket-memory`` and ``dpdk-socket-cores``
configuration options.

.. note::

   Please check that the value of dpdk-socket-memory is large enough to
   accommodate the MTU size being used. For more information please refer to
   `DPDK shared memory calculations`_

The userspace kernel driver can be configured using the ``dpdk-driver``
configuration option. See ``config.yaml`` for more details.

.. note::

   Changing dpdk related configuration options will trigger a restart of
   Open vSwitch, and subsequently interrupt instance connectivity.

Charm configuration
-------------------

The below example bundle excerpt will enable the use of DPDK for an OVN
deployment.

.. code-block:: yaml

   ovn-chassis-dpdk:
     options:
       enable-dpdk: True
       bridge-interface-mappings: br-ex:00:53:00:00:00:42

   ovn-chassis:
     options:
       enable-dpdk: False
       bridge-interface-mappings: br-ex:bond0
       prefer-chassis-as-gw: True

.. caution::

   As configured by the charms, the units configured to use DPDK will not
   participate in the overlay network and will also not be able to provide
   services such as external DHCP to SR-IOV enabled units in the same
   deployment.

   As such it is important to have at least one other named ovn-chassis
   application in the deployment with ``enable-dpdk`` set to 'False' and the
   ``prefer-chassis-as-gw`` configuration option set to 'True'. Doing so will
   inform the CMS (Cloud Management System) that shared services such as
   gateways and external DHCP services should not be scheduled to the
   DPDK-enabled nodes.

DPDK bonding
~~~~~~~~~~~~

Once network interface cards are bound to DPDK they will be invisible to the
standard Linux kernel network stack, and subsequently it is not possible to use
standard system tools to configure bonding.

For DPDK interfaces the charm supports configuring bonding in Open vSwitch.
This is accomplished via the ``dpdk-bond-mappings`` and ``dpdk-bond-config``
configuration options. Example:

.. code-block:: yaml

   ovn-chassis-dpdk:
     options:
       enable-dpdk: True
       bridge-interface-mappings: br-ex:dpdk-bond0
       dpdk-bond-mappings: "dpdk-bond0:00:53:00:00:00:42 dpdk-bond0:00:53:00:00:00:51"
       dpdk-bond-config: ":balance-slb:off:fast"

   ovn-chassis:
     options:
       enable-dpdk: False
       bridge-interface-mappings: br-ex:bond0
       prefer-chassis-as-gw: True

In this example, the network interface cards associated with the two MAC
addresses provided will be used to build a bond identified by a port named
'dpdk-bond0' which will be attached to the 'br-ex' bridge.

DPDK overlay networking setup
-----------------------------

OVN with DPDK can be configured for overlay networking in user space.

Prerequisites
~~~~~~~~~~~~~

Ensure that your system is set up for DPDK in accordance with the
`DPDK setup guide`_
Additionally, you need an external bridge (which is typically named, though the
name can be anything ``br-ex``) configured for Open vSwitch (OVS). If this bridge
is not yet set up, refer to the guide on `creating an OVS bridge`_ The reasoning
for creating this bridge using MAAS is that it allows assigning an IP to it or
one of its VLAN sub-interfaces.

.. note::

    This bridge is very much needed as when running Open vSwitch in user space
    rather than kernel space Open vSwitch. This bridge allows use of the kernel
    network stack for routing and ARP resolution. The datapath needs to look up
    the routing table and ARP table to prepare the tunnel header and transmit
    data to the output port.

New deployments
~~~~~~~~~~~~~~~

MAAS will populate the interfaces key for the Open vSwitch bridge. If you do
``netplan apply`` or ``reboot`` with this configuration after transitioning to DPDK
it will fail due to the interfaces section of the Netplan configuration. One
solution to this is to simply comment out that section.

You should expect your nonworking Netplan configuration to look something like
this:

.. code-block:: yaml

    network:
        bridges:
            br-ex:
                interfaces: [enfs10f1]
                openvswitch: {}
        ...


Once modified, it should look more like this:

.. code-block:: yaml

    network:
        bridges:
            br-ex:
        #       interfaces: [enfs10f1]
                openvswitch: {}
        ...


.. note::

    We aim to add support for configuring DPDK bridges to Netplan and MAAS to
    relieve the user of the task of manually updating the Netplan configuration.

Existing deployments
~~~~~~~~~~~~~~~~~~~~

For existing deployments, you will need to create the additional linked VLAN for
use in overlay space and properly configure the external bridge to transition to
DPDK mode successfully by setting the proper values in the charm config.


Example netplan configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Below is an example of a Netplan configuration to set up ``br-ex`` with a linked
VLAN (``br-ex.42``), while maintaining DPDK compatibility:

.. code-block:: yaml

    network:
        bridges:
            br-ex:
                macaddress: 02:00:5e:00:53:01
                openvswitch: {}
                mtu: 1500

        vlans:
            br-ex.42:
                macaddress: 02:00:5e:00:53:01
                addresses:
                - 203.0.113.11/24
                id: 42
                link: br-ex
                mtu: 1500


If your external bridge is already set up, you will need to configure a linked
VLAN to handle overlay traffic. This can be done by assigning a linked VLAN
interface to the bridge using Netplan as seen above with ``br-ex.42``. Use Netplan
to assign a static IP address to the linked interface, ensuring it persists
across reboots. On a reboot, the ``br-ex`` bridge will be configured with the
assigned IP, and OVN Chassis will automatically enable DPDK mode.

Ensure the linked VLAN interface has the same MAC address as the ``br-ex`` bridge.
This is crucial because the MAC address must match across the interfaces for
proper DPDK operation and persistence during restarts.

Here, we use an example IP reserved for documentation, though the IP configuration
needs to match the space being used for the overlay traffic. You will want to use
the `MAAS reserved range`_ feature to set aside addresses for this purpose as you
cannot typically change the network configuration of a deployed machine in MAAS.

.. LINKS
.. _Nova config drive: https://docs.openstack.org/nova/latest/user/metadata.html#config-drives
.. _DPDK supported hardware page: http://core.dpdk.org/supported/
.. _MAAS: https://maas.io/
.. _Customising instance huge pages allocations: https://docs.openstack.org/nova/latest/admin/huge-pages.html#customizing-instance-huge-pages-allocations
.. _DPDK shared memory calculations: https://docs.openvswitch.org/en/latest/topics/dpdk/memory/#shared-memory-calculations
.. _DPDK setup guide: https://docs.openstack.org/charm-guide/latest/admin/networking/ovn/dpdk.html
.. _Creating an OVS bridge: https://docs.openstack.org/project-deploy-guide/charm-deployment-guide/yoga/install-maas.html#create-ovs-bridge
.. _MAAS reserved range: https://maas.io/docs/how-to-manage-ip-ranges
