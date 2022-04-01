.. _release_notes_trilio_21.06:

============
21.06 Trilio
============

Summary
-------

The 21.06 release of the Trilio charms adds support for new configuration
options introduced in TrilioVault 4.1 Hot Fix 1. This release also introduces
support for using an S3 backend for storing backups.

.. important::

   The Trilio charms are not released with the same cadence as the other
   OpenStack charms. Instead, they are released shortly after releases of the
   Trilio code.

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

Upgrading to 21.06 Trilio charms
--------------------------------

The Trilio charms are upgraded in the usual way:

.. code-block:: none

   juju upgrade-charm trilio-data-mover
   juju upgrade-charm trilio-dm-api
   juju upgrade-charm trilio-horizon-plugin
   juju upgrade-charm trilio-wlm

Configuration option changes for 4.1 HF1
----------------------------------------

Configuration options have been updated in the charms. Any new options
are ignored unless the Trilio packages have been upgraded to 4.1 HF1 or
greater.

.. list-table:: **Configuration options**
   :header-rows: 1
   :widths: 20 26 10 60

   * - Charm
     - Configuration Key
     - Status
     - Explanation

   * - trilio-wlm
     - max-wait-for-upload
     - New
     - The amount of time (in hours) that snapshot upload operation should wait for an upload to complete.

   * - trilio-wlm
     - client-retry-limit
     - New
     - This is the number of times wlm will try to connect to OpenStack services.

   * - trilio-wlm
     - misfire-grace-time
     - New
     - The grace time (in seconds) before the global job scheduler triggers marks a snapshot as missed.

   * - trilio-wlm
     - use-internal-endpoints
     - Changed
     - This is now used when selecting a keystone endpoint for Trilio to use.

   * - trilio-data-mover
     - cinder-http-retries
     - New
     - The number of times datamover will try to connect to the cinder service.

   * - trilio-dm-api
     - dmapi-workers
     - New
     - Number of dmapi workers. This replaces the previous worker-multiplier option.

   * - trilio-dm-api
     - worker-multiplier
     - Removed
     - Replaced by *dmapi-workers*

   * - trilio-dm-api
     - haproxy-queue-timeout
     - Changed
     - Default changed from 9 seconds to 10 minutes.

   * - trilio-dm-api
     - haproxy-connect-timeout
     - Changed
     - Default changed from 9 seconds to 10 minutes.

   * - trilio-dm-api
     - haproxy-client-timeout
     - Changed
     - Default changed from 9 seconds to 10 minutes.

   * - trilio-dm-api
     - haproxy-server-timeout
     - Changed
     - Default changed from 9 seconds to 10 minutes.


Using S3 to store backups
-------------------------

The Trilio charms now support using an S3 compatible storage service to store
backups. This is achieved by setting the ``backup-target-type`` option of the
trilio-data-mover and trilio-wlm charms to `S3` and set the following
configuration options to provide information regarding the S3 service:

* ``tv-s3-endpoint-url`` the URL of the S3 storage (can be omitted if using AWS)
* ``tv-s3-secret-key`` the secret key for accessing the S3 storage
* ``tv-s3-access-key`` the access key for accessing the S3 storage
* ``tv-s3-region-name`` the region for accessing the S3 storage
* ``tv-s3-bucket`` the S3 bucket to use to storage backups in
* ``tv-s3-ssl-cert`` the SSL CA to use when connecting to the S3 service

 See `DeploymentGuideTrilio`_ for more details.

.. warning::

   Switching between S3 and NFS backups types is not supported or tested.


.. LINKS
.. _DeploymentGuideTrilio: https://docs.openstack.org/project-deploy-guide/charm-deployment-guide/latest/app-trilio-vault.html
