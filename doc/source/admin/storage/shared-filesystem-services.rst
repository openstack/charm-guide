==========================
Shared filesystem services
==========================

Overview
--------

As of the 20.02 OpenStack Charms release, with OpenStack Rocky or later,
support for integrating Manila with CephFS to provide shared filesystems is
available.

Three new charms are needed to deploy this solution: 'manila',
'manila-ganesha', and 'ceph-fs'. The 'manila' charm provides the Manila API
service to the OpenStack deployment, the 'ceph-fs' charm provides the Ceph
services required to provide CephFS, and the 'manila-ganesha' charm integrates
these two via a Manila managed NFS gateway (Ganesha) to provide access
controlled NFS mounts to OpenStack instances.

Deployment
----------

.. warning::

   Throughout this guide make sure ``openstack-origin`` matches the value you
   used when `deploying OpenStack`_.

One way to add Manila Ganesha is to do so during the bundle deployment of a new
OpenStack cloud. This is done by means of a bundle overlay, such as
`manila-ganesha-overlay.yaml`:

.. code-block:: yaml

   machines:
     '0':
       series: bionic
     '1':
       series: bionic
     '2':
       series: bionic
     '3':
       series: bionic
   relations:
   - - manila:ha
     - manila-hacluster:ha
   - - manila-ganesha:ha
     - manila-ganesha-hacluster:ha
   - - ceph-mon:mds
     - ceph-fs:ceph-mds
   - - ceph-mon:client
     - manila-ganesha:ceph
   - - manila-ganesha:shared-db
     - percona-cluster:shared-db
   - - manila-ganesha:amqp
     - rabbitmq-server:amqp
   - - manila-ganesha:identity-service
     - keystone:identity-credentials
   - - manila:remote-manila-plugin
     - manila-ganesha:manila-plugin
   - - manila:amqp
     - rabbitmq-server:amqp
   - - manila:identity-service
     - keystone:identity-service
   - - manila:shared-db
     - percona-cluster:shared-db
   series: bionic
   applications:
     ceph-fs:
       charm: cs:ceph-fs
       num_units: 2
       options:
         source: cloud:bionic-train
     manila-hacluster:
       charm: cs:hacluster
     manila-ganesha-hacluster:
       charm: cs:hacluster
     manila-ganesha:
       charm: cs:manila-ganesha
       series: bionic
       num_units: 3
       options:
         openstack-origin: cloud:bionic-train
         vip: <INSERT VIP(S)>
       bindings:
         public: public
         admin: admin
         internal: internal
         shared-db: internal
         amqp: internal
         # This could also be another existing space
         tenant-storage: tenant-storage
       to:
       - 'lxd:1'
       - 'lxd:2'
       - 'lxd:3'
     manila:
       charm: cs:manila
       series: bionic
       num_units: 3
       options:
         openstack-origin: cloud:bionic-train
         vip: <INSERT VIP(S)>
         default-share-backend: cephfsnfs1
         share-protocols: NFS
       bindings:
         public: public
         admin: admin
         internal: internal
         shared-db: internal
         amqp: internal
       to:
       - 'lxd:1'
       - 'lxd:2'
       - 'lxd:3'

.. warning::

   The machine mappings will almost certainly need to be changed.

To deploy OpenStack with Manila Ganesha:

.. code-block:: none

   juju deploy ./base.yaml --overlay ./manila-ganesha-overlay.yaml

Where `base.yaml` is a bundle to deploy OpenStack. See the `Getting started
tutorial`_ for an introduction to bundle usage.

Configuration
-------------

To create and access CephFS shares over NFS, you'll need to `create the share`_
and then you'll need to `grant access`_ to the share.

Spaces
------

This charm can optionally dedicate a provider's physical network to serving
Ganesha NFS shares. It does so through its support for Juju spaces.

The charm uses a space called 'tenant-storage' and it should be accessible
(routed is ok) to all tenants that expect to access the Manila shares. The
easiest way to ensure this access is to create a provider network in OpenStack
that is mapped to the same network layer as this space is. For example, the
storage space is mapped to VLAN 120, then an OpenStack administrator should
create a provider network that maps to the same VLAN. For example:

.. code-block:: none

   openstack network create \
       --provider-network-type vlan \
       --provider-segment 120 \
       --share \
       --provider-physical-network physnet1 \
       tenant-storage

   openstack subnet create tenant \
       --network=tenant-storage \
       --subnet-range 10.1.10.0/22 \
       --gateway 10.1.10.1 \
       --allocation-pool start=10.1.10.50,end=10.1.13.254

When creating the space in MAAS that corresponds to this network, be sure that
DHCP is disabled in this space. If MAAS performs any additional allocations in
this space, ensure that the range configured for the subnet in Neutron does not
overlap with the MAAS subnets.

If dedicating a network space is not desired, it is also possible to use
Ganesha over a routed network. Manila's IP access restrictions will continue to
secure access to Ganesha even for a network that is not managed by Neutron. In
order for the latter to apply, a provider network is required, and guests must
be attached to that provider network.

.. LINKS
.. _deploying OpenStack: install-openstack
.. _create the share: https://docs.openstack.org/manila/latest/admin/cephfs_driver.html#create-cephfs-nfs-share
.. _grant access: https://docs.openstack.org/manila/latest/admin/cephfs_driver.html#allow-access-to-cephfs-nfs-share
.. _Getting started tutorial: https://docs.openstack.org/charm-guide/latest/getting-started/index.html
