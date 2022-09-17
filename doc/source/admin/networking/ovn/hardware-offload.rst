=========================================
Implementing hardware offloading with OVN
=========================================

The OVN chassis can be configured to prepare network interface cards (NICs) for
use with hardware offloading and make them available to OpenStack.

.. note::

   For general information on OVN, refer to the main :doc:`index` page.

.. caution::

   This feature is to be considered Tech Preview. OVN has more stringent
   requirements for match/action support in the hardware than for example
   Neutron ML2+OVS. Make sure to acquire hardware with appropriate support.

   Depending on hardware vendor, it may be required to install third-party
   drivers (DKMS) in order to successfully use this feature.

Hardware offload support makes use of SR-IOV as an underlying mechanism to
accelerate the data path between a virtual machine instance and the NIC
hardware. But as opposed to traditional SR-IOV support the accelerated ports
can be connected to the Open vSwitch integration bridge which allows instances
to take part in regular tenant networks. The NIC also supports hardware
offloading of tunnel encapsulation and de-encapsulation.

With OVN, the Layer 3 routing features are implemented as flow rules in Open
vSwitch. This in turn may allow Layer 3 routing to also be offloaded to NICs
with appropriate driver and firmware support.

Prerequisites
-------------

* Ubuntu 22.04 LTS or later

* Linux kernel >= 5.15

* Open vSwitch 2.17

* OVN 22.03

* OpenStack Yoga or later

Please refer to the :doc:`sriov` page for information on kernel configuration.

Charm configuration
-------------------

The below example bundle excerpt will enable hardware offloading for an OVN
deployment.

.. code-block:: yaml

   applications:

     ovn-chassis:
       charm: ch:ovn-chassis
       channel: $CHANNEL_OVN
       options:
         enable-hardware-offload: true
         sriov-numvfs: "enp3s0f0:32 enp3s0f1:32"

     neutron-api:
       charm: ch:neutron-api
       channel: $CHANNEL_OPENSTACK
       options:
         enable-hardware-offload: true

     nova-compute:
       charm: ch:nova-compute
       channel: $CHANNEL_OPENSTACK
       options:
         pci-passthrough-whitelist: '{"address": "*:03:*", "physical_network": null}'

.. caution::

   After deploying the above example, the machines hosting ovn-chassis
   units must be rebooted for the changes to take effect.

Boot an instance
----------------

OpenStack can now be directed to boot an instance and attach it to a hardware
offloaded port.

First create a port with ``vnic-type`` 'direct' and ``binding-profile`` with
'switchdev' capabilities:

.. code-block:: none

   openstack port create --network my-network --vnic-type direct \
       --binding-profile '{"capabilities": ["switchdev"]}' direct_port1

Then create an instance connected to the newly created port:

.. code-block:: none

   openstack server create --flavor my-flavor --key-name my-key \
       --nic port-id=direct_port1 my-instance

Validate that traffic is offloaded
----------------------------------

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

.. LINKS
.. _traffic control monitor: https://manpages.ubuntu.com/manpages/focal/man8/tc.8.html
.. _data path control tools: https://manpages.ubuntu.com/manpages/focal/man8/ovs-dpctl.8.html
