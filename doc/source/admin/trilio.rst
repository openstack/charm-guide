===========================
TrilioVault data protection
===========================

Overview
--------

`TrilioVault`_ is a data protection solution that integrates with OpenStack. It
allows end-users to back up and restore point-in-time snapshots of their own
workloads (cloud instances). TrilioVault is implemented via the `Trilio
charms`_.

.. note::

   TrilioVault is not part of the OpenStack project. It is a commercially
   supported propriety product.

Prerequisites
-------------

* Ubuntu 18.04 LTS or 20.04 LTS
* OpenStack Queens, Stein, Train, or Ussuri
* an NFS server for snapshot storage or S3 compatible storage
* a license (see the project's homepage)

Supported combinations
----------------------

The below table shows the support status for versions of TrilioVault for each
version of Ubuntu and OpenStack.

.. list-table:: **Supported combinations**
   :header-rows: 1
   :widths: 12 12 20 20

   * - Ubuntu
     - OpenStack
     - TrilioVault 4.0
     - TrilioVault 4.1

   * - Bionic
     - Queens
     - ✔
     - ✔

   * - Bionic
     - Stein
     - ✔
     - ✔

   * - Bionic
     - Train
     - ✔
     - ✔

   * - Bionic
     - Ussuri
     - X
     - ✔

   * - Focal
     - Ussuri
     - X
     - ✔

Deployment
----------

The TrilioVault solution consists of three core services:

* TrilioVault Workload Manager: the main API - used to manage snapshots (e.g.
  creation and restores). This is implemented with the trilio-wlm charm.

* TrilioVault Data Mover: deployed alongside OpenStack Nova - responsible for
  managing the instance snapshot process at the cloud level. This is
  implemented with the trilio-data-mover charm.

* TrilioVault Data Mover API: the API for managing the TrilioVault Data Mover
  services - used by the Workload Manager. This is implemented with the
  trilio-dm-api charm.

An OpenStack Dashboard plugin is also provided, allowing for snapshot
management to take place via a Web UI. This is implemented with the
trilio-horizon-plugin charm.

An overlay bundle is used to deploy to an existing OpenStack cloud. An example
is provided below.

.. note::

   Ensure that the value for ``openstack-origin`` matches the currently
   deployed OpenStack release.

   The ``ceph-mon:client`` relation is only needed if the cloud is already
   configured to use Ceph-backed VM images (via either Cinder or Nova).

.. code-block:: yaml

   series: bionic

   applications:

     trilio-wlm:
       charm: ch:openstack-charmers-trilio-wlm
       num_units: 1
       options:
         openstack-origin: cloud:bionic-train
         triliovault-pkg-source: 'deb [trusted=yes] https://apt.fury.io/triliodata-4-1/ /'

     trilio-data-mover:
       charm: ch:openstack-charmers-trilio-data-mover
       options:
         triliovault-pkg-source: 'deb [trusted=yes] https://apt.fury.io/triliodata-4-1/ /'

     trilio-dm-api:
       charm: ch:openstack-charmers-trilio-dm-api
       num_units: 1
       options:
         openstack-origin: cloud:bionic-train
         triliovault-pkg-source: 'deb [trusted=yes] https://apt.fury.io/triliodata-4-1/ /'

     trilio-horizon-plugin:
       charm: ch:openstack-charmers-trilio-horizon-plugin
       options:
         triliovault-pkg-source: 'deb [trusted=yes] https://apt.fury.io/triliodata-4-1/ /'

   relations:

     - - trilio-horizon-plugin:dashboard-plugin
       - openstack-dashboard:dashboard-plugin
     - - trilio-dm-api:identity-service
       - keystone:identity-service
     - - trilio-dm-api:shared-db
       - percona-cluster:shared-db
     - - trilio-dm-api:amqp
       - rabbitmq-server:amqp
     - - trilio-data-mover:amqp
       - rabbitmq-server:amqp
     - - trilio-data-mover:juju-info
       - nova-compute:juju-info
     - - trilio-wlm:shared-db
       - percona-cluster:shared-db
     - - trilio-wlm:amqp
       - rabbitmq-server:amqp
     - - trilio-wlm:identity-service
       - keystone:identity-service
     - - trilio-data-mover:ceph
       - ceph-mon:client
     - - trilio-data-mover:shared-db
       - percona-cluster:shared-db

.. note::

   The trilio-wlm and trilio-dm-api charms must be deployed with
   ``openstack-origin`` >= 'cloud:bionic-train' - even for Queens deployments.
   These parts of the TrilioVault deployment are Python 3 only and have
   dependency version requirements that are only supported from Train onwards.

Configure storage
-----------------

Once the deployment completes the trilio-wlm and trilio-data-mover applications
will be in a blocked state (see :command:`juju status`). To rectify this, both
applications must be have their workload backup storage configured.

TrilioVault supports NFS and S3 backends for storing workload backups. The
storage type used by TrilioVault is determined by the ``backup-target-type``
configuration option in the trilio-data-mover and trilio-wlm charms.

.. warning::

   Switching between S3 and NFS backups types is not supported or tested.

NFS
~~~

To configure for an NFS backend:

.. code-block:: none

   juju config trilio-data-mover backup-target-type=nfs
   juju config trilio-wlm backup-target-type=nfs

Secondly, point both the trilio-wlm and trilio-data-mover applications to the
same NFS share(s):

.. code-block:: none

   juju config trilio-data-mover nfs-shares=10.40.3.20:/srv/triliovault
   juju config trilio-wlm nfs-shares=10.40.3.20:/srv/triliovault

Multiple NFS shares can be specified by using a comma separated list:

.. code-block:: none

   juju config trilio-data-mover nfs-shares="10.40.3.20:/srv/triliovault,10.40.3.30:/srv/triliovault2"
   juju config trilio-wlm nfs-shares="10.40.3.20:/srv/triliovault,10.40.3.30:/srv/triliovault2"

Mount settings for the NFS shares can be passed via the ``nfs-options``
configuration option in the trilio-wlm and trilio-data-mover charms.

.. code-block:: none

   juju config trilio-data-mover nfs-options="nolock,soft,timeo=180,intr,lookupcache=none"
   juju config trilio-wlm nfs-options="nolock,soft,timeo=180,intr,lookupcache=none"

S3
~~

To configure for an S3 backend:

.. code-block:: none

   juju config trilio-data-mover backup-target-type=s3
   juju config trilio-wlm backup-target-type=s3

Parameters that describe the S3 service are passed with configuration
options available to both the trilio-wlm and trilio-data-mover charms:

* ``tv-s3-endpoint-url`` the URL of the S3 storage (can be omitted if using AWS)
* ``tv-s3-secret-key`` the secret key for accessing the S3 storage
* ``tv-s3-access-key`` the access key for accessing the S3 storage
* ``tv-s3-region-name`` the region for accessing the S3 storage
* ``tv-s3-bucket`` the S3 bucket to use to storage backups in
* ``tv-s3-ssl-cert`` the SSL CA to use when connecting to the S3 service

Options are set to the same value for both applications. For example:

.. code-block:: none

   juju config trilio-data-mover tv-s3-endpoint-url=http://s3.example.com/
   juju config trilio-data-mover tv-s3-secret-key=superSecretKey
   juju config trilio-data-mover tv-s3-access-key=secretAccessKey
   juju config trilio-data-mover tv-s3-region-name=RegionOne
   juju config trilio-data-mover tv-s3-bucket=backups
   juju config trilio-wlm tv-s3-endpoint-url=http://s3.example.com/
   juju config trilio-wlm tv-s3-secret-key=superSecretKey
   juju config trilio-wlm tv-s3-access-key=secretAccessKey
   juju config trilio-wlm tv-s3-region-name=RegionOne
   juju config trilio-wlm tv-s3-bucket=backups

The required parameters are dependent upon the given S3 service,
making the setting of some charm options unnecessary.

Authorisation
-------------

The TrilioVault service account must be granted the authorisation to access
resources from across users and projects to perform backups. This will involve
providing it with the cloud's admin password (set up by the keystone
application). This is done with the trilio-wlm charm's
``create-cloud-admin-trust`` action:

.. code-block:: none

   juju run-action --wait trilio-wlm/leader create-cloud-admin-trust password=cloudadminpassword

Licensing
---------

The TrilioVault deployment must be licensed. This is done by uploading the
license file (attaching it as a charm resource) and running the trilio-wlm
charm's ``create-license`` action:

.. code-block:: none

   juju attach trilio-wlm license=mycorp_tv.lic
   juju run-action trilio-wlm/leader create-license

The trilio-wlm and trilio-data-mover applications should now be in the 'active'
state and ready for use.

.. LINKS
.. _TrilioVault: https://www.trilio.io/triliovault-for-openstack-2/
.. _Trilio charms: https://opendev.org/openstack?tab=&sort=recentupdate&q=charm-trilio
