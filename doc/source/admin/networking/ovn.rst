==========================
Open Virtual Network (OVN)
==========================

Overview
--------

Open Virtual Network (OVN) can be deployed to provide networking services as
part of an OpenStack cloud.

.. note::

   There are feature `gaps from ML2/OVS`_ and deploying legacy ML2/OVS with
   the OpenStack Charms is still available.

OVN charms:

* neutron-api-plugin-ovn
* ovn-central
* ovn-chassis
* ovn-dedicated-chassis

.. note::

   OVN is supported by Charmed OpenStack starting with OpenStack Train. OVN is
   the default configuration in the `OpenStack Base bundle`_ reference
   implementation.

   Instructions for migrating legacy clouds to OVN are found on the
   :doc:`cdg:ovn-migration` page.

Deployment
----------

OVN makes use of Public Key Infrastructure (PKI) to authenticate and authorize
control plane communication. The charm requires a Certificate Authority to be
present in the model as represented by the ``certificates`` relation.
Certificates must be managed by Vault.

.. note::

   For Vault deployment instructions see the `vault charm`_. For certificate
   management information read the :doc:`cdg:app-certificate-management`
   page of the Deploy Guide.

To deploy OVN:

.. code-block:: none

   juju config neutron-api manage-neutron-plugin-legacy-mode=false

   juju deploy neutron-api-plugin-ovn
   juju deploy ovn-central -n 3 --config source=cloud:bionic-ussuri
   juju deploy ovn-chassis

   juju add-relation neutron-api-plugin-ovn:certificates vault:certificates
   juju add-relation neutron-api-plugin-ovn:neutron-plugin \
       neutron-api:neutron-plugin-api-subordinate
   juju add-relation neutron-api-plugin-ovn:ovsdb-cms ovn-central:ovsdb-cms
   juju add-relation ovn-central:certificates vault:certificates
   juju add-relation ovn-chassis:ovsdb ovn-central:ovsdb
   juju add-relation ovn-chassis:certificates vault:certificates
   juju add-relation ovn-chassis:nova-compute nova-compute:neutron-plugin

The OVN components used for the data plane is deployed by the ovn-chassis
subordinate charm. A subordinate charm is deployed together with a principle
charm, nova-compute in the example above.

If you require a dedicated software gateway you may deploy the data plane
components as a principle charm through the use of the
`ovn-dedicated-chassis charm`_.

.. note::

   For a concrete example take a look at the `OpenStack Base bundle`_.

High availability
-----------------

OVN is HA by design; take a look at the `OVN section of the Infrastructure high
availability`_ page.

.. _ovn_configuration:

Configuration
-------------

OVN integrates with OpenStack through the OVN ML2 driver. On OpenStack Ussuri
and onwards the OVN ML2 driver is maintained as an in-tree driver in Neutron.
On OpenStack Train it is maintained separately as per the `networking-ovn
plugin`_.

General Neutron configuration is still done through the `neutron-api charm`_,
and the subset of configuration specific to OVN is done through the
`neutron-api-plugin-ovn charm`_.

Hardware offloading support
~~~~~~~~~~~~~~~~~~~~~~~~~~~

It is possible to configure chassis to prepare network interface cards (NICs)
for use with hardware offloading and make them available to OpenStack.

.. caution::

   This feature is to be considered Tech Preview. OVN has more stringent
   requirements for match/action support in the hardware than for example
   Neutron ML2+OVS. Make sure to acquire hardware with appropriate support.
   
   Depending on hardware vendor, it may be required to install third party
   drivers (DKMS) in order to successfully use this feature.

Hardware offload support makes use of SR-IOV as an underlying mechanism to
accelerate the data path between a virtual machine instance and the NIC
hardware. But as opposed to traditional SR-IOV support the accelerated ports
can be connected to the Open vSwitch integration bridge which allows instances
to take part in regular tenant networks. The NIC also supports hardware
offloading of tunnel encapsulation and decapsulation.

With OVN the Layer3 routing features are implemented as flow rules in Open
vSwitch. This in turn may allow Layer 3 routing to also be offloaded to NICs
with appropriate driver and firmware support.

Prerequisites
^^^^^^^^^^^^^

* Ubuntu 22.04 LTS or later

* Linux kernel >= 5.15

* Open vSwitch 2.17

* OVN 22.03

* OpenStack Yoga or later

Please refer to the `SR-IOV for networking support`_ section for information on
kernel configuration.

Charm configuration
^^^^^^^^^^^^^^^^^^^

The below example bundle excerpt will enable hardware offloading for an OVN
deployment.

.. code-block:: yaml

   applications:
     ovn-chassis:
       charm: cs:ovn-chassis
       options:
         enable-hardware-offload: true
         sriov-numvfs:  "enp3s0f0:32 enp3s0f1:32"
     neutron-api:
       charm: cs:neutron-api
       options:
         enable-hardware-offload: true
     nova-compute:
       charm: cs:nova-compute
       options:
         pci-passthrough-whitelist: '{"address": "*:03:*", "physical_network": null}'

.. caution::

   After deploying the above example the machines hosting ovn-chassis
   units must be rebooted for the changes to take effect.

Boot an instance
^^^^^^^^^^^^^^^^

Now we can tell OpenStack to boot an instance and attach it to an hardware
offloaded port. This must be done in two stages, first we create a port with
``vnic-type`` 'direct' and ``binding-profile`` with 'switchdev' capabilities.
Then we create an instance connected to the newly created port:

.. code-block:: none

   openstack port create --network my-network --vnic-type direct \
       --binding-profile '{"capabilities": ["switchdev"]}' direct_port1
   openstack server create --flavor my-flavor --key-name my-key \
       --nic port-id=direct_port1 my-instance

Validate that traffic is offloaded
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The `traffic control monitor`_ command can be used to observe updates to
filters which is one of the mechanisms used to program the NIC switch hardware.
Look for the 'in_hw' and 'not_in_hw' labels.

.. code-block:: none

   sudo tc monitor

.. code-block:: console

   replaced filter dev eth62 ingress protocol ip pref 3 flower chain 0 handle 0x9
     dst_mac fa:16:3e:b2:20:82
     src_mac fa:16:3e:b9:db:c8
     eth_type ipv4
     ip_proto tcp
     ip_tos 67deeb90
     dst_ip 10.42.0.17/28
     tcp_flags 22
     ip_flags nofrag
     in_hw
       action order 1: tunnel_key set
       src_ip 0.0.0.0
       dst_ip 10.6.12.8
       key_id 4
       dst_port 6081
       csum pipe
       index 15 ref 1 bind 1

       action order 2: mirred (Egress Redirect to device genev_sys_6081) stolen
       index 18 ref 1 bind 1
       cookie d4885b4d38419f7fd7ae77a11bc78b0b

Open vSwitch has a rich set of tools to monitor traffic flows and you can use
the `data path control tools`_ to monitor offloaded flows.

.. code-block:: none

   sudo ovs-appctl dpctl/dump-flows type=offloaded

.. code-block:: console

   tunnel(tun_id=0x4,src=10.6.12.3,dst=10.6.12.7,tp_dst=6081,geneve({class=0x102,type=0x80,len=4,0x20007/0x7fffffff}),flags(+key)),recirc_id(0),in_port(2),eth(src=fa:16:3e:f8:52:5c,dst=00:00:00:00:00:00/01:00:00:00:00:00),eth_type(0x0800),ipv4(proto=6,frag=no),tcp_flags(psh|ack), packets:2, bytes:204, used:5.710s, actions:7
   tunnel(tun_id=0x4,src=10.6.12.3,dst=10.6.12.7,tp_dst=6081,geneve({class=0x102,type=0x80,len=4,0x20007/0x7fffffff}),flags(+key)),recirc_id(0),in_port(2),eth(src=fa:16:3e:f8:52:5c,dst=00:00:00:00:00:00/01:00:00:00:00:00),eth_type(0x0800),ipv4(proto=6,frag=no),tcp_flags(ack), packets:3, bytes:230, used:5.710s, actions:7
   tunnel(tun_id=0x4,src=10.6.12.8,dst=10.6.12.7,tp_dst=6081,geneve({class=0x102,type=0x80,len=4,0x60007/0x7fffffff}),flags(+key)),recirc_id(0),in_port(2),eth(src=fa:16:3e:b2:20:82,dst=00:00:00:00:00:00/01:00:00:00:00:00),eth_type(0x0800),ipv4(proto=6,frag=no),tcp_flags(syn|ack), packets:0, bytes:0, used:6.740s, actions:7
   tunnel(tun_id=0x4,src=10.6.12.8,dst=10.6.12.7,tp_dst=6081,geneve({class=0x102,type=0x80,len=4,0x60007/0x7fffffff}),flags(+key)),recirc_id(0),in_port(2),eth(src=fa:16:3e:b2:20:82,dst=00:00:00:00:00:00/01:00:00:00:00:00),eth_type(0x0800),ipv4(proto=6,frag=no),tcp_flags(ack), packets:180737, bytes:9400154, used:0.000s, actions:7
   recirc_id(0),in_port(6),eth(src=26:8a:07:82:a7:2f,dst=01:80:c2:00:00:0e),eth_type(0x88cc), packets:5, bytes:990, used:14.340s, actions:drop
   recirc_id(0),in_port(7),eth(src=fa:16:3e:b9:db:c8,dst=fa:16:3e:b2:20:82),eth_type(0x0800),ipv4(dst=10.42.0.16/255.255.255.240,proto=6,tos=0/0x3,frag=no),tcp_flags(syn), packets:0, bytes:0, used:6.910s, actions:set(tunnel(tun_id=0x4,dst=10.6.12.8,ttl=64,tp_dst=6081,key6(bad key length 1, expected 0)(01)geneve({class=0x102,type=0x80,len=4,0x70006}),flags(key))),2
   recirc_id(0),in_port(7),eth(src=fa:16:3e:b9:db:c8,dst=fa:16:3e:b2:20:82),eth_type(0x0800),ipv4(dst=10.42.0.16/255.255.255.240,proto=6,tos=0/0x3,frag=no),tcp_flags(ack), packets:935904, bytes:7504070178, used:0.590s, actions:set(tunnel(tun_id=0x4,dst=10.6.12.8,ttl=64,tp_dst=6081,key6(bad key length 1, expected 0)(01)geneve({class=0x102,type=0x80,len=4,0x70006}),flags(key))),2
   recirc_id(0),in_port(7),eth(src=fa:16:3e:b9:db:c8,dst=fa:16:3e:b2:20:82),eth_type(0x0800),ipv4(dst=10.42.0.16/255.255.255.240,proto=6,tos=0/0x3,frag=no),tcp_flags(psh|ack), packets:3873, bytes:31053714, used:0.590s, actions:set(tunnel(tun_id=0x4,dst=10.6.12.8,ttl=64,tp_dst=6081,key6(bad key length 1, expected 0)(01)geneve({class=0x102,type=0x80,len=4,0x70006}),flags(key))),2


SR-IOV for networking support
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Single root I/O virtualization (SR-IOV) enables splitting a single physical
network port into multiple virtual network ports known as virtual functions
(VFs). The division is done at the PCI level which allows attaching the VF
directly to a virtual machine instance, bypassing the networking stack of the
hypervisor hosting the instance.

The main use case for this feature is to support applications with high
bandwidth requirements. For such applications the normal plumbing through the
userspace virtio driver in QEMU will consume too much resources from the host.

It is possible to configure chassis to prepare network interface cards (NICs)
for use with SR-IOV and make them available to OpenStack.

Prerequisites
^^^^^^^^^^^^^

To use the feature you need to use a NIC with support for SR-IOV.

Machines need to be pre-configured with appropriate kernel command-line
parameters. The charm does not handle this facet of configuration and it is
expected that the user configure this either manually or through the bare metal
provisioning layer (for example `MAAS`_). Example:

.. code-block:: none

   intel_iommu=on iommu=pt probe_vf=0

Charm configuration
^^^^^^^^^^^^^^^^^^^

Enable SR-IOV, map physical network name 'physnet2' to the physical port named
'enp3s0f0' and create 4 virtual functions on it:

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

Add mapping between physical network name, physical port and Open vSwitch
bridge:

.. code-block:: none

   juju config ovn-chassis ovn-bridge-mappings=physnet2:br-ex
   juju config ovn-chassis bridge-interface-mappings br-ex:a0:36:9f:dd:37:a8

.. note::

   The above configuration allows OVN to configure an 'external' port on one
   of the chassis for providing DHCP and metadata to instances connected
   directly to the network through SR-IOV.

For OpenStack to make use of the VFs the ``neutron-sriov-agent`` needs to talk
to RabbitMQ:

.. code:: bash

   juju add-relation ovn-chassis:amqp rabbitmq-server:amqp

OpenStack Nova also needs to know which PCI devices it is allowed to pass
through to instances:

.. code:: bash

   juju config nova-compute pci-passthrough-whitelist='{"vendor_id":"8086", "product_id":"10ed", "physical_network":"physnet2"}'

Boot an instance
^^^^^^^^^^^^^^^^

Now we can tell OpenStack to boot an instance and attach it to an SR-IOV port.
This must be done in two stages, first we create a port with ``vnic-type``
'direct' and then we create an instance connected to the newly created port:

.. code:: bash

   openstack port create --network my-network --vnic-type direct my-port
   openstack server create --flavor my-flavor --key-name my-key \
      --nic port-id=my-port my-instance

DPDK support
~~~~~~~~~~~~

It is possible to configure chassis to use experimental DPDK userspace network
acceleration.

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

Internal DNS resolution
~~~~~~~~~~~~~~~~~~~~~~~

OVN supports Neutron internal DNS resolution. To configure this:

.. caution::

   At the time of this writing the internal DNS support does not include
   reverse lookup (PTR-records) of instance IP addresses, only forward lookup
   (A and AAAA-records) of instance names. This is tracked in `LP #1857026`_.

.. code::

   juju config neutron-api enable-ml2-dns=true
   juju config neutron-api dns-domain=openstack.example.
   juju config neutron-api-plugin-ovn dns-servers="1.1.1.1 8.8.8.8"

.. note::

   The value for the ``dns-domain`` configuration option must
   not be set to 'openstack.local.' as that will effectively disable the
   feature.

   It is also important to end the string with a '.' (dot).

When you set ``enable-ml2-dns`` to 'true' and set a value for ``dns-domain``,
Neutron will add details such as instance name and DNS domain name to each
individual Neutron port associated with instances. The OVN ML2 driver will
populate the ``DNS`` table of the Northbound and Southbound databases:

.. code::

   # ovn-sbctl list DNS
   _uuid               : 2e149fa8-d27f-4106-99f5-a08f60c443bf
   datapaths           : [b25ed99a-89f1-49cc-be51-d215aa6fb073]
   external_ids        : {dns_id="4c79807e-0755-4d17-b4bc-eb57b93bf78d"}

   records             : {"c-1"="192.0.2.239", "c-1.openstack.example"="192.0.2.239"}

On the chassis, OVN creates flow rules to redirect UDP port 53 packets (DNS)
to the local ``ovn-controller`` process:

.. code::

   cookie=0xdeaffed, duration=77.575s, table=22, n_packets=0, n_bytes=0, idle_age=77, priority=100,udp6,metadata=0x2,tp_dst=53 actions=controller(userdata=00.00.00.06.00.00.00.00.00.01.de.10.00.00.00.64,pause),resubmit(,23)
   cookie=0xdeaffed, duration=77.570s, table=22, n_packets=0, n_bytes=0, idle_age=77, priority=100,udp,metadata=0x2,tp_dst=53 actions=controller(userdata=00.00.00.06.00.00.00.00.00.01.de.10.00.00.00.64,pause),resubmit(,23)

The local ``ovn-controller`` process then decides if it should respond to the
DNS query directly or if it needs to be forwarded to the real DNS server.

External connectivity
~~~~~~~~~~~~~~~~~~~~~

Interface and network to bridge mapping is done through the
`ovn-chassis charm`_.

OVN provides a more flexible way of configuring external Layer3 networking than
the legacy ML2+DVR configuration as OVN does not require every node
(``Chassis`` in OVN terminology) in a deployment to have direct external
connectivity. This plays nicely with Layer3-only datacenter fabrics (RFC 7938).

East/West traffic is distributed by default. North/South traffic is highly
available by default. Liveness detection is done using the Bidirectional
Forwarding Detection (BFD) protocol.

Networks for use with external Layer3 connectivity should have mappings on
chassis located in the vicinity of the datacenter border gateways. Having two
or more chassis with mappings for a Layer3 network will have OVN automatically
configure highly available routers with liveness detection provided by the
Bidirectional Forwarding Detection (BFD) protocol.

Chassis without direct external mapping to a external Layer3 network will
forward traffic through a tunnel to one of the chassis acting as a gateway for
that network.

Networks for use with external Layer2 connectivity should have mappings present
on all chassis with potential to host the consuming payload.

.. note::

   It is not necessary nor recommended to add mapping for external
   Layer3 networks to all chassis. Doing so will create a scaling problem at
   the physical network layer that needs to be resolved with globally shared
   Layer2 (does not scale) or tunneling at the top-of-rack switch layer (adds
   complexity) and is generally not a recommended configuration.

Example configuration with explicit bridge-interface-mappings:

.. code:: bash

   juju config neutron-api flat-network-providers=physnet1
   juju config ovn-chassis ovn-bridge-mappings=physnet1:br-provider
   juju config ovn-chassis \
       bridge-interface-mappings='br-provider:00:00:5e:00:00:42 \
                                  br-provider:00:00:5e:00:00:51'
   openstack network create --external --share --provider-network-type flat \
                            --provider-physical-network physnet1 ext-net
   openstack subnet create --network ext-net \
                           --subnet-range 192.0.2.0/24 \
                           --no-dhcp --gateway 192.0.2.1 \
                           ext

It is also possible to influence the scheduling of routers on a per named
ovn-chassis application basis. The benefit of this method is that you do not
need to provide MAC addresses when configuring Layer3 connectivity in the
charm. For example:

.. code-block:: none

   juju config ovn-chassis-border \
      ovn-bridge-mappings=physnet1:br-provider \
      bridge-interface-mappings=br-provider:bond0 \
      prefer-chassis-as-gw=true

   juju config ovn-chassis \
      ovn-bridge-mappings=physnet1:br-provider \
      bridge-interface-mappings=br-provider:bond0 \

In the above example units of the ovn-chassis-border application with
appropriate bridge mappings will be eligible for router scheduling.

Usage
-----

Create networks, routers and subnets through the OpenStack API or CLI as you
normally would.

The OVN ML2 driver will translate the OpenStack network constructs into high
level logical rules in the OVN Northbound database.

The ``ovn-northd`` daemon in turn translates this into data in the Southbound
database.

The local ``ovn-controller`` daemon on each chassis consumes these rules and
programs flows in the local Open vSwitch database.

Information queries
~~~~~~~~~~~~~~~~~~~

The OVN databases are configured to use the `Clustered Database Service
Model`_. In this configuration only the leader processes transactions and the
administrative client tools are configured to require a connection to the
leader to operate.

The leader of the Northbound and Southbound databases does not have to coincide
with the charm leader, so before querying databases you must consult the output
of :command:`juju status` to check which unit is the leader of the database you
want to query. Example:

.. code-block:: none

   juju status ovn-central

.. code-block:: console

   Unit            Workload  Agent  Machine  Public address  Ports              Message
   ovn-central/0*  active    idle   0/lxd/5  10.246.114.39   6641/tcp,6642/tcp  Unit is ready (leader: ovnnb_db)
   ovn-central/1   active    idle   1/lxd/4  10.246.114.15   6641/tcp,6642/tcp  Unit is ready (northd: active)
   ovn-central/2   active    idle   2/lxd/2  10.246.114.27   6641/tcp,6642/tcp  Unit is ready (leader: ovnsb_db)

In the above example 'ovn-central/0' is the leader for the Northbound DB,
'ovn-central/1' has the active ``ovn-northd`` daemon and 'ovn-central/2' is the
leader for the Southbound DB.

OVSDB Cluster status
^^^^^^^^^^^^^^^^^^^^

The cluster status as conveyed through :command:`juju status` is updated each
time a hook is run, in some circumstances it may be necessary to get an
immediate view of the current cluster status.

To get an immediate view of the database clusters:

.. code-block:: none

   juju run --application ovn-central 'ovn-appctl -t \
       /var/run/ovn/ovnnb_db.ctl cluster/status OVN_Northbound'
   juju run --application ovn-central 'ovn-appctl -t \
       /var/run/ovn/ovnsb_db.ctl cluster/status OVN_Southbound'

Querying DBs
^^^^^^^^^^^^

To query the individual databases:

.. code-block:: none

   juju run --unit ovn-central/0 'ovn-nbctl show'
   juju run --unit ovn-central/2 'ovn-sbctl show'
   juju run --unit ovn-central/2 'ovn-sbctl lflow-list'

As an alternative you may provide the administrative client tools with
command-line arguments for path to certificates and IP address of servers so
that you can run the client from anywhere:

.. code-block:: none

   ovn-nbctl \
      -p /etc/ovn/key_host \
      -C /etc/ovn/ovn-central.crt \
      -c /etc/ovn/cert_host \
      --db ssl:10.246.114.39:6641,ssl:10.246.114.15:6641,ssl:10.246.114.27:6641 \
      show

Note that for remote administrative write access to the Southbound DB you must
use port number '16642'. This is due to OVN RBAC being enabled on the standard
'6642' port:

.. code-block:: none

   ovn-sbctl \
      -p /etc/ovn/key_host \
      -C /etc/ovn/ovn-central.crt \
      -c /etc/ovn/cert_host \
      --db ssl:10.246.114.39:16642,ssl:10.246.114.15:16642,ssl:10.246.114.27:16642 \
      show

Data plane flow tracing
^^^^^^^^^^^^^^^^^^^^^^^

SSH into one of the chassis units to get access to various diagnostic tools:

.. code-block:: none

   juju ssh ovn-chassis/0

   sudo ovs-vsctl show

   sudo ovs-ofctl -O OpenFlow13 dump-flows br-int

   sudo ovs-appctl -t ovs-vswitchd \
      ofproto/trace br-provider \
      in_port=enp3s0f0,icmp,nw_src=192.0.2.1,nw_dst=192.0.2.100

   sudo ovn-trace \
      -p /etc/ovn/key_host \
      -C /etc/ovn/ovn-chassis.crt \
      -c /etc/ovn/cert_host \
      --db ssl:10.246.114.39:6642,ssl:10.246.114.15:6642,ssl:10.246.114.27:6642 \
      --ovs ext-net 'inport=="provnet-dde76bc9-0620-44f7-b99a-99cfc66e1095" && \
      eth.src==30:e1:71:5c:7a:b5 && \
      eth.dst==fa:16:3e:f7:15:73 && \
      ip4.src==10.172.193.250 && \
      ip4.dst==10.246.119.8 && \
      icmp4.type==8 && \
      ip.ttl == 64'

.. note::

   OVN makes use of OpenFlow 1.3 or newer and as such the charm configures
   bridges to use these protocols. To be able to successfully use the
   :command:`ovs-ofctl` command you must specify the OpenFlow version as shown
   in the example above.

   You may issue the :command:`ovs-vsctl list bridge` command to show what
   protocols are enabled on the bridges.

.. LINKS
.. _vault charm: https://jaas.ai/vault/
.. _Toward Convergence of ML2+OVS+DVR and OVN: http://specs.openstack.org/openstack/neutron-specs/specs/ussuri/ml2ovs-ovn-convergence.html
.. _ovn-dedicated-chassis charm: https://jaas.ai/u/openstack-charmers/ovn-dedicated-chassis/
.. _networking-ovn plugin: https://docs.openstack.org/networking-ovn/latest/
.. _neutron-api charm: https://jaas.ai/neutron-api/
.. _neutron-api-plugin-ovn charm: https://jaas.ai/u/openstack-charmers/neutron-api-plugin-ovn/
.. _ovn-chassis charm: https://jaas.ai/u/openstack-charmers/ovn-chassis/
.. _OpenStack Base bundle: https://github.com/openstack-charmers/openstack-bundles/tree/master/development/openstack-base-bionic-ussuri-ovn
.. _gaps from ML2/OVS: https://docs.openstack.org/neutron/latest/ovn/gaps.html
.. _OVN section of the Infrastructure high availability: https://docs.openstack.org/charm-guide/latest/admin/ha.html#ovn
.. _OpenStack Charms Deployment Guide: https://docs.openstack.org/project-deploy-guide/charm-deployment-guide/latest/
.. _Nova config drive: https://docs.openstack.org/nova/latest/user/metadata.html#config-drives
.. _DPDK supported hardware page: http://core.dpdk.org/supported/
.. _MAAS: https://maas.io/
.. _Customizing instance huge pages allocations: https://docs.openstack.org/nova/latest/admin/huge-pages.html#customizing-instance-huge-pages-allocations
.. _traffic control monitor: http://manpages.ubuntu.com/manpages/focal/man8/tc.8.html#monitor
.. _data path control tools: http://manpages.ubuntu.com/manpages/focal/man8/ovs-dpctl.8.html
.. _Clustered Database Service Model: http://docs.openvswitch.org/en/latest/ref/ovsdb.7/#clustered-database-service-model
.. _DPDK shared memory calculations: https://docs.openvswitch.org/en/latest/topics/dpdk/memory/#shared-memory-calculations

.. BUGS
.. _LP #1857026: https://bugs.launchpad.net/ubuntu/+source/ovn/+bug/1857026
