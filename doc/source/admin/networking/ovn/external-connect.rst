=========================================
Setting up external connectivity with OVN
=========================================

OVN provides a more flexible way of configuring external Layer 3 networking
than the legacy ML2+DVR configuration. This is because every chassis is not
required to have direct external connectivity. This plays nicely with Layer
3-only datacentre fabrics (RFC 7938).

.. note::

   For general information on OVN, refer to the main :doc:`index` page.

East/West traffic is distributed by default. North/South traffic is highly
available by default. Liveness detection is done using the Bidirectional
Forwarding Detection (BFD) protocol.

Networks for use with external Layer 3 connectivity should have mappings on
chassis located in the vicinity of the datacentre border gateways. Having two
or more chassis with mappings for a Layer 3 network will have OVN automatically
configure highly available routers with liveness detection provided by the
Bidirectional Forwarding Detection (BFD) protocol.

Chassis without direct external mapping to a external Layer 3 network will
forward traffic through a tunnel to one of the chassis acting as a gateway for
that network.

Networks for use with external Layer 2 connectivity should have mappings present
on all chassis with potential to host the consuming payload.

.. note::

   It is not necessary nor recommended to add mapping for external Layer 3
   networks to all chassis. Doing so will create a scaling problem at the
   physical network layer that needs to be resolved with globally shared Layer2
   (does not scale) or tunneling at the top-of-rack switch layer (adds
   complexity) and is generally not a recommended configuration.

Example configuration with explicit bridge-interface-mappings:

.. code-block:: none

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
need to provide MAC addresses when configuring Layer 3 connectivity in the
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
