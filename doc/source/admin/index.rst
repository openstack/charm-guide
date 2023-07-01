=============
How-to guides
=============

Operations
----------

General cloud operations:

.. toctree::
   :maxdepth: 1

   ops-change-keystone-password
   ops-scale-back-nova-compute
   ops-unseal-vault
   ops-config-tls-vault-api
   ops-live-migrate-vms
   ops-live-migrate-routers
   ops-replace-vault-node
   ops-scale-out-nova-compute
   ops-start-innodb-from-outage
   ops-auto-glance-image-updates
   ops-implement-ha-with-vip
   ops-cloud-admin-access
   ops-reissue-tls-certs
   ops-replace-rabbitmq-node
   ops-repair-rabbitmq-node
   ops-restart-partitioned-rabbitmq-cluster
   ops-replace-control-plane-service-ha
   ops-replace-hyperconverged-compute-node
   ops-scale-back-ovn-central
   ops-remove-mysql8-node
   ops-show-extended-server-attributes

Ceph storage operations (published in the Charmed Ceph documentation):

.. toctree::
   :maxdepth: 1

   Adding OSDs <https://ubuntu.com/ceph/docs/adding-osds>
   Removing OSDs <https://ubuntu.com/ceph/docs/removing-osds>
   Adding MONs <https://ubuntu.com/ceph/docs/adding-mons>
   Replacing OSD disks <https://ubuntu.com/ceph/docs/replacing-osd-disks>
   Encryption at Rest <https://ubuntu.com/ceph/docs/encryption-at-rest>
   Software upgrades <https://ubuntu.com/ceph/docs/software-upgrades>

.. _howto_management:

Management
----------

Cloud management how-to guides:

.. toctree::
   :maxdepth: 1

   managing-power-events
   deferred-events
   policy-overrides
   ha
   instance-ha
   vtpm
   vgpu
   trilio

Upgrades
~~~~~~~~

.. toctree::
   :maxdepth: 1

   upgrades/overview
   upgrades/charms
   upgrades/series
   upgrades/openstack

Compute
~~~~~~~

.. toctree::
   :maxdepth: 1

   compute/ironic
   compute/nova-cells
   compute/pci-passthrough
   compute/nfv

Storage
~~~~~~~

.. toctree::
   :maxdepth: 1

   storage/encryption-at-rest
   storage/ceph-rgw-multisite
   storage/ceph-rbd-mirror
   storage/ceph-erasure-coding
   storage/shared-filesystem-services
   storage/cinder-replication
   storage/swift

Networking
~~~~~~~~~~

.. toctree::
   :maxdepth: 1

   networking/ovn/index
   networking/hardware-offloading
   networking/load-balancing
   networking/interface-config

Security
~~~~~~~~

.. toctree::
   :maxdepth: 1

   security/keystone
   security/tls
