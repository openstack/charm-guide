==========================
Shared filesystem services
==========================

Overview
--------

The `Manila`_ project can be integrated with `CephFS`_ to provide shared
filesystem services. To add such services to a Ceph-backed OpenStack cloud,
three charms are needed:

* ceph-fs
* manila
* manila-ganesha

The manila charm provides the Manila API service, the ceph-fs charm exposes the
Ceph services required by CephFS, and the manila-ganesha charm integrates
Manila and CephFS via a Manila-managed NFS gateway (Ganesha) to provide
access-controlled NFS mounts to OpenStack VMs.

Deployment
----------

The below overlay bundle encapsulates what is needed in terms of the
deployment.

.. important::

   An overlay's parameters should be adjusted as per the local environment
   (e.g. the machine mappings). In particular, the following placeholders must
   be replaced with actual values:

   * ``$SERIES``
   * ``$OPENSTACK_ORIGIN``
   * ``$VIP``
   * ``$CHANNEL_CEPH``
   * ``$CHANNEL_HACLUSTER``
   * ``$CHANNEL_OPENSTACK``

   Replace ``$SERIES`` with the Ubuntu release running on the cloud nodes (e.g.
   'jammy'). For ``$OPENSTACK_ORIGIN`` and ``$VIP`` see the corresponding charm
   options.  For channel information see the
   :doc:`../../project/charm-delivery` page.

.. code-block:: yaml

   series: $SERIES

   machines:
     '0':
     '1':
     '2':
     '3':

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

   applications:

     ceph-fs:
       charm: ch:ceph-fs
       channel: $CHANNEL_CEPH
       num_units: 2
       options:
         source: $OPENSTACK_ORIGIN

     manila-hacluster:
       charm: ch:hacluster
       channel: $CHANNEL_HACLUSTER

     manila-ganesha-hacluster:
       charm: ch:hacluster
       channel: $CHANNEL_OPENSTACK

     manila-ganesha:
       charm: ch:manila-ganesha
       channel: $CHANNEL_OPENSTACK
       num_units: 3
       options:
         openstack-origin: $OPENSTACK_ORIGIN
         vip: $VIP
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
       charm: ch:manila
       channel: $CHANNEL_OPENSTACK
       num_units: 3
       options:
         openstack-origin: $OPENSTACK_ORIGIN
         vip: $VIP
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

The feature should be deployable during (or after) the deployment of a cloud
- as per the Juju documentation: `How to add an overlay bundle`_.

Configuration
-------------

To create and access CephFS shares over NFS, first create the share and then
grant access to the share. See the following upstream Manila resources for
guidance:

* `Create CephFS NFS share`_
* `Allow access to CephFS NFS share`_

Dedicated physical network
~~~~~~~~~~~~~~~~~~~~~~~~~~

The manila-ganesha charm can optionally dedicate a provider's physical network
to serving Ganesha NFS shares.

The charm uses a network space called 'tenant-storage' and it should be
accessible to all tenants that expect to access the Manila shares. The easiest
way to ensure this access is to create a provider network in OpenStack that is
mapped to the same network layer as the space is.

For example, if the space is mapped to VLAN 120, then a provider network can be
created that maps to the same VLAN:

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

.. note::

   As an alternative to using a network space, Ganesha can be used over a
   routed network. Manila's IP access restrictions will continue to secure
   access to Ganesha even for a network that is not managed by Neutron;
   however, a provider network is required, and guests must be attached to it.

.. LINKS
.. _Manila: https://docs.openstack.org/manila/latest/
.. _CephFS: https://docs.ceph.com/en/latest/cephfs/
.. _Create CephFS NFS share: https://docs.openstack.org/manila/latest/admin/cephfs_driver.html#create-cephfs-nfs-share
.. _Allow access to CephFS NFS share: https://docs.openstack.org/manila/latest/admin/cephfs_driver.html#allow-access-to-cephfs-nfs-share
.. _How to add an overlay bundle: https://juju.is/docs/sdk/add-an-overlay-bundle
