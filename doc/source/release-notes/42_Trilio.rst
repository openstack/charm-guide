.. _release_notes_trilio_4.2:

==========
4.2 Trilio
==========

Summary
-------

The 4.2 release of the Trilio charms adds support for TrilioVault 4.2.

.. important::

   The Trilio charms are not released with the same cadence as the other
   OpenStack charms. Instead, they are released after releases of the
   Trilio code.

Supported combinations
----------------------

The below table shows the support status for versions of TrilioVault for each
version of Ubuntu and OpenStack.

.. list-table:: **Supported combinations**
   :header-rows: 1
   :widths: 12 12 20 20 20

   * - Ubuntu
     - OpenStack
     - TrilioVault 4.0
     - TrilioVault 4.1
     - TrilioVault 4.2

   * - Bionic
     - Queens
     - ✔
     - ✔
     - ✔

   * - Bionic
     - Stein
     - ✔
     - ✔
     - ✔

   * - Bionic
     - Train
     - ✔
     - ✔
     - ✔

   * - Bionic
     - Ussuri
     - X
     - ✔
     - ✔

   * - Focal
     - Ussuri
     - X
     - ✔
     - ✔

   * - Focal
     - Wallaby
     - X
     - X
     - ✔

Upgrading to 4.2 Trilio charms
------------------------------

The Trilio charms are now managed in channels within the `CharmHub`_. The
Trilio charms have a track which corresponds to the version of Trilio they
are compatable with, so each charm has a 4.0, 4.1 and 4.2 channel. The charms
are also compatable with the previous version of Trilio so that they can
provide an upgrade path.

For example if an existing deployment is using Trilio 4.1 then the first step
is to upgrade to the 4.2 charms which will provide backward compatability with
4.1.

.. code-block:: none

   juju refresh --channel 4.2/stable trilio-dm-api
   juju refresh --channel 4.2/stable trilio-wlm
   juju refresh --channel 4.2/stable trilio-data-mover
   juju refresh --channel 4.2/stable trilio-horizon-plugin

Once all charms have been upgraded the Trilio packages can be upgraded to
4.2.

.. code-block:: none

   juju config trilio-dm-api triliovault-pkg-source='deb [trusted=yes] https://apt.fury.io/triliodata-4-2/ /'
   juju config trilio-wlm triliovault-pkg-source='deb [trusted=yes] https://apt.fury.io/triliodata-4-2/ /'
   juju config trilio-data-mover triliovault-pkg-source='deb [trusted=yes] https://apt.fury.io/triliodata-4-2/ /'
   juju config trilio-horizon-plugin triliovault-pkg-source='deb [trusted=yes] https://apt.fury.io/triliodata-4-2/ /'


Configuration option changes for 4.2
------------------------------------

Configuration options have been updated in the charms. Any new options
are ignored unless the Trilio packages have been upgraded to 4.2 or
greater.

.. list-table:: **Configuration options**
   :header-rows: 1
   :widths: 20 26 10 60

   * - Charm
     - Configuration Key
     - Status
     - Explanation

   * - trilio-wlm
     - progress-tracking-update-interval
     - New
     - Seconds to wait for tracking file updated before contego crash called

   * - trilio-wlm
     - process-timeout:
     - New
     - Process timeout in seconds, used in file-search tool.

   * - trilio-data-mover
     - vault_s3_max_pool_connections:
     - New
     - The maximum number of connections to keep in a connection pool

   * - trilio-data-mover
     - bucket_object_lock:
     - New
     - S3 bucket object locking is enabled

   * - trilio-data-mover
     - use_manifest_suffix:
     - New
     - When set to False, s3fuse creates manifest file names without suffix

   * - trilio-data-mover
     - multipath-rescan-opts:
     - New
     - Comma separated options which are needed to form a rescan command like

   * - trilio-data-mover
     - nbd-timeout:
     - New
     - Amount of time, in seconds, to wait for NBD device start up

Deploying Trilio 4.2 charms
---------------------------

For information on deploying Trilio charms for the first time see
 `DeploymentGuideTrilio`_ for more details.

.. LINKS
.. _DeploymentGuideTrilio: https://docs.openstack.org/charm-guide/latest/admin/trilio.html
.. _CharmHub: https://charmhub.io/
