:orphan:

.. _cinder_replication_dr:

=============================================
Cinder volume replication - Disaster recovery
=============================================

Overview
--------

This is the disaster recovery scenario of a Cinder volume replication
deployment. It should be read in conjunction with the :doc:`Cinder volume
replication <cinder-replication>` page.

Scenario description
--------------------

Disaster recovery involves an uncontrolled failover to the secondary site.
Site-b takes over from a troubled site-a and becomes the de-facto primary site,
which includes writes to its images. Control is passed back to site-a once it
is repaired.

.. warning::

   The charms support the underlying OpenStack servcies in their native ability
   to failover and failback. However, a significant degree of administrative
   care is still needed in order to ensure a successful recovery.

   For example,

   * primary volume images that are currently in use may experience difficulty
     during their demotion to secondary status

   * running VMs will lose connectivity to their volumes

   * subsequent image resyncs may not be straightforward

   Any work necessary to rectify data issues resulting from an uncontrolled
   failover is beyond the scope of the OpenStack charms and this document.

Simulation
----------

For the sake of understanding some of the rudimentary aspects involved in
disaster recovery a simulation is provided.

Preparation
~~~~~~~~~~~

Create the replicated data volume and confirm it is available:

.. code-block:: none

   openstack volume create --size 5 --type site-a-repl vol-site-a-repl-data
   openstack volume list

Simulate a failure in site-a by turning off all of its Ceph MON daemons:

.. code-block:: none

   juju ssh site-a-ceph-mon/0 sudo systemctl stop ceph-mon.target
   juju ssh site-a-ceph-mon/1 sudo systemctl stop ceph-mon.target
   juju ssh site-a-ceph-mon/2 sudo systemctl stop ceph-mon.target

Modify timeout and retry settings
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When a Ceph cluster fails communication between Cinder and the failed cluster
will be interrupted and the RBD driver will accommodate with retries and
timeouts.

To accelerate the failover mechanism, timeout and retry settings on the
cinder-ceph unit in site-a can be modified:

.. code-block:: none

   juju ssh cinder-ceph-a/0
   > sudo apt install -y crudini
   > sudo crudini --set /etc/cinder/cinder.conf cinder-ceph-a rados_connect_timeout 1
   > sudo crudini --set /etc/cinder/cinder.conf cinder-ceph-a rados_connection_retries 1
   > sudo crudini --set /etc/cinder/cinder.conf cinder-ceph-a rados_connection_interval 0
   > sudo crudini --set /etc/cinder/cinder.conf cinder-ceph-a replication_connect_timeout 1
   > sudo systemctl restart cinder-volume
   > exit

These configuration changes are only intended to be in effect during the
failover transition period. They should be reverted afterwards since the
default values are fine for normal operations.

Failover
~~~~~~~~

Perform the failover of site-a, confirm its cinder-volume host is disabled, and
that the volume remains available:

.. code-block:: none

   cinder failover-host cinder@cinder-ceph-a
   cinder service-list
   openstack volume list

Confirm that the Cinder log file (``/var/log/cinder/cinder-volume.log``) on
unit ``cinder/0`` contains the successful failover message: ``Failed over to
replication target successfully.``.

Revert timeout and retry settings
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Revert the configuration changes made to the cinder-ceph backend:

.. code-block:: none

   juju ssh cinder-ceph-a/0
   > sudo crudini --del /etc/cinder/cinder.conf cinder-ceph-a rados_connect_timeout
   > sudo crudini --del /etc/cinder/cinder.conf cinder-ceph-a rados_connection_retries
   > sudo crudini --del /etc/cinder/cinder.conf cinder-ceph-a rados_connection_interval
   > sudo crudini --del /etc/cinder/cinder.conf cinder-ceph-a replication_connect_timeout
   > sudo systemctl restart cinder-volume
   > exit

Write to the volume
~~~~~~~~~~~~~~~~~~~

Create a VM (named 'vm-with-data-volume'):

.. code-block:: none

   openstack server create --image focal-amd64 --flavor m1.tiny \
      --key-name mykey --network int_net vm-with-data-volume

   FLOATING_IP=$(openstack floating ip create -f value -c floating_ip_address ext_net)
   openstack server add floating ip vm-with-data-volume $FLOATING_IP

Attach the volume to the VM, write some data to it, and detach it:

.. code-block:: none

   openstack server add volume vm-with-data-volume vol-site-a-repl-data

   ssh -i ~/cloud-keys/mykey ubuntu@$FLOATING_IP
   > sudo mkfs.ext4 /dev/vdc
   > mkdir data
   > sudo mount /dev/vdc data
   > sudo chown ubuntu: data
   > echo "This is a test." > data/test.txt
   > sync
   > sudo umount /dev/vdc
   > exit

   openstack server remove volume vm-with-data-volume vol-site-a-repl-data

Repair site-a
~~~~~~~~~~~~~

In the current example, site-a is repaired by starting the Ceph MON daemons:

.. code-block:: none

   juju ssh site-a-ceph-mon/0 sudo systemctl start ceph-mon.target
   juju ssh site-a-ceph-mon/1 sudo systemctl start ceph-mon.target
   juju ssh site-a-ceph-mon/2 sudo systemctl start ceph-mon.target

Confirm that the MON cluster is now healthy (it may take a while):

.. code-block:: none

   juju status site-a-ceph-mon

   Unit                       Workload  Agent  Machine  Public address  Ports  Message
   site-a-ceph-mon/0          active    idle   14       10.5.0.15              Unit is ready and clustered
   site-a-ceph-mon/1*         active    idle   15       10.5.0.31              Unit is ready and clustered
   site-a-ceph-mon/2          active    idle   16       10.5.0.11              Unit is ready and clustered

Image resync
~~~~~~~~~~~~

Putting site-a back online at this point will lead to two primary images for
each replicated volume. This is a split-brain condition that cannot be resolved
by the RBD mirror daemon. Hence, before failback is invoked each replicated
volume will need a resync of its images (site-b images are more recent than the
site-a images).

The image resync is a two-step process that is initiated on the ceph-rbd-mirror
unit in site-a:

Demote the site-a images with the ``demote`` action:

.. code-block:: none

   juju run-action --wait site-a-ceph-rbd-mirror/0 demote pools=cinder-ceph-a

Flag the site-a images for a resync with the ``resync-pools`` action. The
``pools`` argument should point to the corresponding site's pool, which by
default is the name of the cinder-ceph application for the site (here
'cinder-ceph-a'):

.. code-block:: none

   juju run-action --wait site-a-ceph-rbd-mirror/0 resync-pools i-really-mean-it=true pools=cinder-ceph-a

The Ceph RBD mirror daemon will perform the resync in the background.

Failback
~~~~~~~~

Prior to failback, confirm that the images of all replicated volumes in site-a
are fully synchronised. Perform a check with the ceph-rbd-mirror charm's
``status`` action as per :ref:`RBD image status <rbd_image_status>`:

.. code-block:: none

   juju run-action --wait site-a-ceph-rbd-mirror/0 status verbose=true | grep -A3 volume-

This will take a while.

The state and description for site-a images will transition to:

.. code-block:: console

        state:       up+syncing
        description: bootstrapping, IMAGE_SYNC/CREATE_SYNC_POINT

The intermediate values will look like:

.. code-block:: console

        state:       up+replaying
        description: replaying, {"bytes_per_second":110318.93,"entries_behind_primary":4712.....

The final values, as expected, will become:

.. code-block:: console

        state:       up+replaying
        description: replaying, {"bytes_per_second":0.0,"entries_behind_primary":0.....

The failback of site-a can now proceed:

.. code-block:: none

   cinder failover-host cinder@cinder-ceph-a --backend_id default

Confirm the original health of Cinder services (as per :ref:`Cinder service
list <cinder_service_list>`):

.. code-block:: none

   cinder service-list

Verification
~~~~~~~~~~~~

Re-attach the volume to the VM and verify that the secondary device contains
the expected data:

.. code-block:: none

   openstack server add volume vm-with-data-volume vol-site-a-repl-data
   ssh -i ~/cloud-keys/mykey ubuntu@$FLOATING_IP
   > sudo mount /dev/vdc data
   > cat data/test.txt
   This is a test.

We can also check the status of the image as per :ref:`RBD image status
<rbd_image_status>` to verify that the primary indeed resides in site-a again:

.. code-block:: none

   juju run-action --wait site-a-ceph-rbd-mirror/0 status verbose=true | grep -A3 volume-

   volume-c44d4d20-6ede-422a-903d-588d1b0d51b0:
     global_id:   3a4aa755-c9ee-4319-8ba4-fc494d20d783
     state:       up+stopped
     description: local image is primary
