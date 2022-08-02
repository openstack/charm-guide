======================
Enabling DPDK with OVN
======================

It is possible to configure chassis to use experimental DPDK userspace network
acceleration.

.. note::

   Please see the :doc:`index` page for general information on using OVN with
   Charmed OpenStack.

.. note::

   Currently instances are required to be attached to a external network (also
   known as provider network) for connectivity.  OVN supports distributed DHCP
   for provider networks.  For OpenStack workloads use of `Nova config drive`_
   is required to provide metadata to instances.

Prerequisites
^^^^^^^^^^^^^

To use the feature you need to use a supported CPU architecture and network
interface card (NIC) hardware. Please consult the `DPDK supported hardware
page`_.

Machines need to be pre-configured with appropriate kernel command-line
parameters. The charm does not handle this facet of configuration and it is
expected that the user configure this either manually or through the bare metal
provisioning layer (for example `MAAS`_).

Example:

.. code:: bash

   default_hugepagesz=1G hugepagesz=1G hugepages=64 intel_iommu=on iommu=pt

For the communication between the host userspace networking stack and the guest
virtual NIC driver to work the instances need to be configured to use
hugepages. For OpenStack this can be accomplished by `Customizing instance huge
pages allocations`_.

Example:

.. code:: bash

   openstack flavor set m1.large --property hw:mem_page_size=large

By default, the charm will configure Open vSwitch/DPDK to consume one processor
core + 1G of RAM from each NUMA node on the unit being deployed. This can be
tuned using the ``dpdk-socket-memory`` and ``dpdk-socket-cores`` configuration
options.

.. note::

    Please check that the value of dpdk-socket-memory is large enough to
    accommodate the MTU size being used. For more information please refer to
    `DPDK shared memory calculations`_

The userspace kernel driver can be configured using the ``dpdk-driver``
configuration option. See config.yaml for more details.

.. note::

   Changing dpdk related configuration options will trigger a restart of
   Open vSwitch, and subsequently interrupt instance connectivity.

Charm configuration
^^^^^^^^^^^^^^^^^^^

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
............

Once Network interface cards are bound to DPDK they will be invisible to the
standard Linux kernel network stack and subsequently it is not possible to use
standard system tools to configure bonding.

For DPDK interfaces the charm supports configuring bonding in Open vSwitch.
This is accomplished through the ``dpdk-bond-mappings`` and
``dpdk-bond-config`` configuration options. Example:

.. code:: yaml

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

.. LINKS
.. _Nova config drive: https://docs.openstack.org/nova/latest/user/metadata.html#config-drives
.. _DPDK supported hardware page: http://core.dpdk.org/supported/
.. _MAAS: https://maas.io/
.. _Customizing instance huge pages allocations: https://docs.openstack.org/nova/latest/admin/huge-pages.html#customizing-instance-huge-pages-allocations
.. _DPDK shared memory calculations: https://docs.openvswitch.org/en/latest/topics/dpdk/memory/#shared-memory-calculations
