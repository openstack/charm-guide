=========================
Cinder volume replication
=========================

.. important::

   This page has been identified as being affected by the breaking changes
   introduced between versions 2.9.x and 3.x of the Juju client. Read
   support note :ref:`juju_29_3x_changes` before continuing.

Overview
--------

Cinder volume replication is a primary/secondary failover solution based on
two-way :doc:`ceph-rbd-mirror`.

Deployment
----------

The cloud deployment in this document is based on the stable `openstack-base`_
bundle in the `openstack-bundles`_ repository. The necessary documentation is
found in the `bundle README`_.

A custom overlay bundle (:doc:`cinder-replication-overlay`) is used to extend
the base cloud in order to implement volume replication.

.. note::

   The key elements for adding volume replication to Ceph RBD mirroring is the
   relation between cinder-ceph in one site and ceph-mon in the other (using the
   ``ceph-replication-device`` endpoint) and the cinder-ceph charm
   configuration option ``rbd-mirroring-mode=image``.

Cloud notes:

* The cloud used in these instructions is based on Ubuntu 20.04 LTS (Focal) and
  OpenStack Victoria. The openstack-base bundle may have been updated since.
* The two Ceph clusters are named 'site-a' and 'site-b' and are placed in the
  same Juju model.
* A site's pool is named after its corresponding cinder-ceph application (e.g.
  'cinder-ceph-a' for site-a) and is mirrored to the other site. Each site will
  therefore have two pools: 'cinder-ceph-a' and 'cinder-ceph-b'.
* Glance is only backed by site-a.

To deploy:

.. code-block:: none

   juju deploy ./bundle.yaml --overlay ./cinder-volume-replication-overlay.yaml

Configuration and verification of the base cloud
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Configure the base cloud as per the referenced documentation.

Before proceeding, verify the base cloud by creating a VM and connecting to it
over SSH. See the main bundle's README for guidance.

.. important::

   A known issue affecting the interaction of the ceph-rbd-mirror charm and
   Ceph itself gives the impression of a fatal error. The symptom is messaging
   that appears in :command:`juju status` command output: ``Pools WARNING (1)
   OK (1) Images unknown (1)``. This remains a cosmetic issue however. See bug
   `LP #1892201`_ for details.

Cinder volume types
-------------------

For each site, create replicated and non-replicated Cinder volumes types. A
type is referenced at volume-creation time in order to specify whether the
volume is replicated (or not) and what pool it will reside in.

Type 'site-a-repl' denotes replication in site-a:

.. code-block:: none

   openstack volume type create site-a-repl \
      --property volume_backend_name=cinder-ceph-a \
      --property replication_enabled='<is> True'

Type 'site-a-local' denotes non-replication in site-a:

.. code-block:: none

   openstack volume type create site-a-local \
      --property volume_backend_name=cinder-ceph-a

Type 'site-b-repl' denotes replication in site-b:

.. code-block:: none

   openstack volume type create site-b-repl \
      --property volume_backend_name=cinder-ceph-b \
      --property replication_enabled='<is> True'

Type 'site-b-local' denotes non-replication in site-b:

.. code-block:: none

   openstack volume type create site-b-local \
      --property volume_backend_name=cinder-ceph-b

List the volume types:

.. code-block:: none

   openstack volume type list
   +--------------------------------------+--------------+-----------+
   | ID                                   | Name         | Is Public |
   +--------------------------------------+--------------+-----------+
   | ee70dfd9-7b97-407d-a860-868e0209b93b | site-b-local | True      |
   | b0f6d6b5-9c76-4967-9eb4-d488a6690712 | site-b-repl  | True      |
   | fc89ca9b-d75a-443e-9025-6710afdbfd5c | site-a-local | True      |
   | 780980dc-1357-4fbd-9714-e16a79df252a | site-a-repl  | True      |
   | d57df78d-ff27-4cf0-9959-0ada21ce86ad | __DEFAULT__  | True      |
   +--------------------------------------+--------------+-----------+

.. note::

   In this document, site-b volume types will not be used. They are created
   here for the more generalised case where new volumes may be needed while
   site-a is in a failover state. In such a circumstance, any volumes created
   in site-b will naturally not be replicated (in site-a).

.. _rbd_image_status:

RBD image status
----------------

The status of the two RBD images associated with a replicated volume can be
queried using the ``status`` action of the ceph-rbd-mirror unit for each site.

A state of ``up+replaying`` in combination with the presence of
``"entries_behind_primary":0`` in the image description means the image in one
site is in sync with its counterpart in the other site.

A state of ``up+syncing`` indicates that the sync process is still underway.

A description of ``local image is primary`` means that the image is the
primary.

Consider the volume below that is created and given the volume type of
'site-a-repl'. Its primary will be in site-a and its non-primary (secondary)
will be in site-b:

.. code-block:: none

   openstack volume create --size 5 --type site-a-repl vol-site-a-repl

Their statuses can be queried in each site as shown:

Site a (primary),

.. code-block:: none

   juju run --wait site-a-ceph-rbd-mirror/0 status verbose=true | grep -A3 volume-
         volume-c44d4d20-6ede-422a-903d-588d1b0d51b0:
           global_id:   f66140a6-0c09-478c-9431-4eb1eb16ca86
           state:       up+stopped
           description: local image is primary

Site b (secondary is in sync with the primary),

.. code-block:: none

   juju run --wait site-b-ceph-rbd-mirror/0 status verbose=true | grep -A3 volume-
         volume-c44d4d20-6ede-422a-903d-588d1b0d51b0:
           global_id:   f66140a6-0c09-478c-9431-4eb1eb16ca86
           state:       up+replaying
           description: replaying, {"bytes_per_second":0.0,"entries_behind_primary":0,.....

.. _cinder_service_list:

Cinder service list
-------------------

To verify the state of Cinder services the ``cinder service-list`` command is
used:

.. code-block:: none

   cinder service-list
   +------------------+----------------------+------+---------+-------+----------------------------+---------+-----------------+---------------+
   | Binary           | Host                 | Zone | Status  | State | Updated_at                 | Cluster | Disabled Reason | Backend State |
   +------------------+----------------------+------+---------+-------+----------------------------+---------+-----------------+---------------+
   | cinder-scheduler | cinder               | nova | enabled | up    | 2021-04-08T15:59:25.000000 | -       | -               |               |
   | cinder-volume    | cinder@cinder-ceph-a | nova | enabled | up    | 2021-04-08T15:59:24.000000 | -       | -               | up            |
   | cinder-volume    | cinder@cinder-ceph-b | nova | enabled | up    | 2021-04-08T15:59:25.000000 | -       | -               | up            |
   +------------------+----------------------+------+---------+-------+----------------------------+---------+-----------------+---------------+

Each of the below examples ends with a failback to site-a. The above output is
the desired result.

The failover of a particular site entails the referencing of its corresponding
cinder-volume service host (e.g. ``cinder@cinder-ceph-a`` for site-a). We'll
see how to do this later on.

.. note::

   'cinder-ceph-a' and 'cinder-ceph-b' correspond to the two applications
   deployed via the `cinder-ceph`_ charm. The express purpose of this charm is
   to connect Cinder to a Ceph cluster. See the
   :doc:`cinder-replication-overlay` bundle for details.

Failover, volumes, images, and pools
------------------------------------

This section will show the basics of failover/failback, non-replicated vs
replicated volumes, and what pools are used for the volume images.

In site-a, create one non-replicated and one replicated data volume and list
them:

.. code-block:: none

   openstack volume create --size 5 --type site-a-local vol-site-a-local
   openstack volume create --size 5 --type site-a-repl vol-site-a-repl

   openstack volume list
   +--------------------------------------+------------------+-----------+------+-------------+
   | ID                                   | Name             | Status    | Size | Attached to |
   +--------------------------------------+------------------+-----------+------+-------------+
   | fba13395-62d1-468e-9b9a-40bebd0373e8 | vol-site-a-local | available |    5 |             |
   | c21a539e-d524-4f4d-991b-9b9476d4f930 | vol-site-a-repl  | available |    5 |             |
   +--------------------------------------+------------------+-----------+------+-------------+

Pools and images
~~~~~~~~~~~~~~~~

For 'vol-site-a-local' there should be one image in the 'cinder-ceph-a' pool of
site-a.

For 'vol-site-a-repl' there should be two images: one in the 'cinder-ceph-a'
pool of site-a and one in the 'cinder-ceph-a' pool of site-b:

This can all be confirmed by querying a Ceph MON in each site:

.. code-block:: none

   juju ssh site-a-ceph-mon/0 sudo rbd ls -p cinder-ceph-a

   volume-fba13395-62d1-468e-9b9a-40bebd0373e8
   volume-c21a539e-d524-4f4d-991b-9b9476d4f930

   juju ssh site-b-ceph-mon/0 sudo rbd ls -p cinder-ceph-a

   volume-c21a539e-d524-4f4d-991b-9b9476d4f930

Failover
~~~~~~~~

Perform the failover of site-a:

.. code-block:: none

   cinder failover-host cinder@cinder-ceph-a

Wait until the failover is complete:

.. code-block:: none

   cinder service-list
   +------------------+----------------------+------+----------+-------+----------------------------+---------+-----------------+---------------+
   | Binary           | Host                 | Zone | Status   | State | Updated_at                 | Cluster | Disabled Reason | Backend State |
   +------------------+----------------------+------+----------+-------+----------------------------+---------+-----------------+---------------+
   | cinder-scheduler | cinder               | nova | enabled  | up    | 2021-04-08T17:11:56.000000 | -       | -               |               |
   | cinder-volume    | cinder@cinder-ceph-a | nova | disabled | up    | 2021-04-08T17:11:56.000000 | -       | failed-over     | -             |
   | cinder-volume    | cinder@cinder-ceph-b | nova | enabled  | up    | 2021-04-08T17:11:56.000000 | -       | -               | up            |
   +------------------+----------------------+------+----------+-------+----------------------------+---------+-----------------+---------------+

A failover triggers the promotion of one site and the demotion of the other
(site-b and site-a respectively in this example). Communication between Cinder
and each Ceph cluster is therefore ideal, as in this example.

Inspection
~~~~~~~~~~

By consulting the volume list we see that the replicated volume is still
available but that the non-replicated volume has errored:

.. code-block:: none

   openstack volume list
   +--------------------------------------+------------------+-----------+------+-------------+
   | ID                                   | Name             | Status    | Size | Attached to |
   +--------------------------------------+------------------+-----------+------+-------------+
   | fba13395-62d1-468e-9b9a-40bebd0373e8 | vol-site-a-local | error     |    5 |             |
   | c21a539e-d524-4f4d-991b-9b9476d4f930 | vol-site-a-repl  | available |    5 |             |
   +--------------------------------------+------------------+-----------+------+-------------+

Generally a failover indicates a significant degree of non-confidence in the
primary site, site-a in this case. Once a **local** volume goes into an error
state due to a failover it is expected to not recover after failback. The
errored local volumes should normally be discarded (deleted).

Failback
~~~~~~~~

Failback site-a and confirm the original health of Cinder services (as per
`Cinder service list`_):

.. code-block:: none

   cinder failover-host cinder@cinder-ceph-a --backend_id default
   cinder service-list

Examples
--------

The following two examples will be considered. They will both use replication
and involve the failing over of site-a to site-b:

#. `Data volume used by a VM`_
#. `Bootable volume used by a VM`_

Data volume used by a VM
~~~~~~~~~~~~~~~~~~~~~~~~

In this example, a replicated data volume will be created in site-a and
attached to a VM. The volume's block device will then have some test data
written to it. This will allow for verification of the replicated data once
failover has occurred and the volume is re-attached to the VM.

Preparation
^^^^^^^^^^^

Create the replicated data volume:

.. code-block:: none

   openstack volume create --size 5 --type site-a-repl vol-site-a-repl-data
   openstack volume list
   +--------------------------------------+---------------------------+-----------+------+-------------+
   | ID                                   | Name                      | Status    | Size | Attached to |
   +--------------------------------------+---------------------------+-----------+------+-------------+
   | f23732c1-3257-4e58-a214-085c460abf56 | vol-site-a-repl-data      | available |    5 |             |
   +--------------------------------------+---------------------------+-----------+------+-------------+

Create the VM (named 'vm-with-data-volume'):

.. code-block:: none

   openstack server create --image focal-amd64 --flavor m1.tiny \
      --key-name mykey --network int_net vm-with-data-volume

   FLOATING_IP=$(openstack floating ip create -f value -c floating_ip_address ext_net)
   openstack server add floating ip vm-with-data-volume $FLOATING_IP

   openstack server list
   +--------------------------------------+----------------------+--------+---------------------------------+--------------------------+---------+
   | ID                                   | Name                 | Status | Networks                        | Image                    | Flavor  |
   +--------------------------------------+----------------------+--------+---------------------------------+--------------------------+---------+
   | fbe07fea-731e-4973-8455-c8466be72293 | vm-with-data-volume  | ACTIVE | int_net=192.168.0.38, 10.5.1.28 | focal-amd64              | m1.tiny |
   +--------------------------------------+----------------------+--------+---------------------------------+--------------------------+---------+

Attach the data volume to the VM:

.. code-block:: none

   openstack server add volume vm-with-data-volume vol-site-a-repl-data

Prepare the block device and write the test data to it:

.. code-block:: none

   ssh -i ~/cloud-keys/mykey ubuntu@$FLOATING_IP
   > sudo mkfs.ext4 /dev/vdc
   > mkdir data
   > sudo mount /dev/vdc data
   > sudo chown ubuntu: data
   > echo "This is a test." > data/test.txt
   > sync
   > exit

Failover
^^^^^^^^

When both sites are online, as is here, it is not recommended to perform a
failover when volumes are in use. This is because Cinder will try to demote the
Ceph image from the primary site, and if there is an active connection to it
the operation may fail (i.e. the volume will transition to an error state).

Here we ensure the volume is not in use by unmounting the block device and
removing it from the VM:

.. code-block:: none

   ssh -i ~/cloud-keys/mykey ubuntu@$FLOATING_IP sudo umount /dev/vdc
   openstack server remove vm-with-data-volume vol-site-a-repl-data

Prior to failover the images of all replicated volumes must be fully
synchronised. Perform a check with the ceph-rbd-mirror charm's ``status``
action as per `RBD image status`_. If the volumes were created in site-a then
the ceph-rbd-mirror unit in site-b is the target:

.. code-block:: none

   juju run --wait site-b-ceph-rbd-mirror/0 status verbose=true | grep -A3 volume-

If all images look good, perform the failover of site-a:

.. code-block:: none

   cinder failover-host cinder@cinder-ceph-a
   cinder service-list
   +------------------+----------------------+------+----------+-------+----------------------------+---------+-----------------+---------------+
   | Binary           | Host                 | Zone | Status   | State | Updated_at                 | Cluster | Disabled Reason | Backend State |
   +------------------+----------------------+------+----------+-------+----------------------------+---------+-----------------+---------------+
   | cinder-scheduler | cinder               | nova | enabled  | up    | 2021-04-08T19:30:29.000000 | -       | -               |               |
   | cinder-volume    | cinder@cinder-ceph-a | nova | disabled | up    | 2021-04-08T19:30:28.000000 | -       | failed-over     | -             |
   | cinder-volume    | cinder@cinder-ceph-b | nova | enabled  | up    | 2021-04-08T19:30:28.000000 | -       | -               | up            |
   +------------------+----------------------+------+----------+-------+----------------------------+---------+-----------------+---------------+

Verification
^^^^^^^^^^^^

Re-attach the volume to the VM:

.. code-block:: none

   openstack server add volume vm-with-data-volume vol-site-a-repl-data

Verify that the secondary device contains the expected data:

.. code-block:: none

   ssh -i ~/cloud-keys/mykey ubuntu@$FLOATING_IP
   > sudo mount /dev/vdc /data
   > cat /data/test.txt
   This is a test.

Failback
^^^^^^^^

Failback site-a and confirm the original health of Cinder services (as per
`Cinder service list`_):

.. code-block:: none

   cinder failover-host cinder@cinder-ceph-a --backend_id default
   cinder service-list

Bootable volume used by a VM
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In this example, a bootable volume will be created in site-a and have a
newly-created VM use that volume as its root device. Identically to the
previous example, the volume's block device will have test data written to it
to use for verification purposes.

Preparation
^^^^^^^^^^^

Create the replicated bootable volume:

.. code-block:: none

   openstack volume create --size 5 --type site-a-repl --image focal-amd64 --bootable vol-site-a-repl-boot

Wait for the volume to become available (it may take a while):

.. code-block:: none

   openstack volume list
   +--------------------------------------+----------------------+-----------+------+-------------+
   | ID                                   | Name                 | Status    | Size | Attached to |
   +--------------------------------------+----------------------+-----------+------+-------------+
   | c44d4d20-6ede-422a-903d-588d1b0d51b0 | vol-site-a-repl-boot | available |    5 |             |
   +--------------------------------------+----------------------+-----------+------+-------------+

Create a VM (named 'vm-with-boot-volume') by specifying the newly-created
bootable volume:

.. code-block:: none

   openstack server create --volume vol-site-a-repl-boot --flavor m1.tiny \
      --key-name mykey --network int_net vm-with-boot-volume

   FLOATING_IP=$(openstack floating ip create -f value -c floating_ip_address ext_net)
   openstack server add floating ip vm-with-boot-volume $FLOATING_IP

   openstack server list
   +--------------------------------------+---------------------+--------+---------------------------------+--------------------------+---------+
   | ID                                   | Name                | Status | Networks                        | Image                    | Flavor  |
   +--------------------------------------+---------------------+--------+---------------------------------+--------------------------+---------+
   | c0a152d7-376b-4500-95d4-7c768a3ff280 | vm-with-boot-volume | ACTIVE | int_net=192.168.0.75, 10.5.1.53 | N/A (booted from volume) | m1.tiny |
   +--------------------------------------+---------------------+--------+---------------------------------+--------------------------+---------+

Write the test data to the block device:

.. code-block:: none

   ssh -i ~/cloud-keys/mykey ubuntu@$FLOATING_IP
   > echo "This is a test." > test.txt
   > sync
   > exit

Failover
^^^^^^^^

As explained previously, when both sites are functional, prior to failover the
replicated volume should not be in use. Since the testing of the replicated
boot volume requires the VM to be rebuilt anyway (Cinder needs to give the
updated Ceph connection credentials to Nova) the easiest way forward is to
simply delete the VM:

.. code-block:: none

   openstack server delete vm-with-boot-volume

Like before, prior to failover, confirm that the images of all replicated
volumes in site-b are fully synchronised. Perform a check with the
ceph-rbd-mirror charm's ``status`` action as per `RBD image status`_:

.. code-block:: none

   juju run --wait site-b-ceph-rbd-mirror/0 status verbose=true | grep -A3 volume-

If all images look good, perform the failover of site-a:

.. code-block:: none

   cinder failover-host cinder@cinder-ceph-a
   cinder service-list
   +------------------+----------------------+------+----------+-------+----------------------------+---------+-----------------+---------------+
   | Binary           | Host                 | Zone | Status   | State | Updated_at                 | Cluster | Disabled Reason | Backend State |
   +------------------+----------------------+------+----------+-------+----------------------------+---------+-----------------+---------------+
   | cinder-scheduler | cinder               | nova | enabled  | up    | 2021-04-08T21:29:12.000000 | -       | -               |               |
   | cinder-volume    | cinder@cinder-ceph-a | nova | disabled | up    | 2021-04-08T21:29:12.000000 | -       | failed-over     | -             |
   | cinder-volume    | cinder@cinder-ceph-b | nova | enabled  | up    | 2021-04-08T21:29:11.000000 | -       | -               | up            |
   +------------------+----------------------+------+----------+-------+----------------------------+---------+-----------------+---------------+

Verification
^^^^^^^^^^^^

Re-create the VM:

.. code-block:: none

   openstack server create --volume vol-site-a-repl-boot --flavor m1.tiny \
      --key-name mykey --network int_net vm-with-boot-volume

   FLOATING_IP=$(openstack floating ip create -f value -c floating_ip_address ext_net)
   openstack server add floating ip vm-with-boot-volume $FLOATING_IP

Verify that the root device contains the expected data:

.. code-block:: none

   ssh -i ~/cloud-keys/mykey ubuntu@$FLOATING_IP
   > cat test.txt
   This is a test.
   > exit

Failback
^^^^^^^^

Failback site-a and confirm the original health of Cinder services (as per
`Cinder service list`_):

.. code-block:: none

   cinder failover-host cinder@cinder-ceph-a --backend_id default
   cinder service-list

Disaster recovery
-----------------

An uncontrolled failover is known as the disaster recovery scenario. It is
characterised by the sudden failure of the primary Ceph cluster. See the
:ref:`Cinder volume replication - Disaster recovery <cinder_replication_dr>`
page for more information.

.. LINKS
.. _openstack-base: https://github.com/openstack-charmers/openstack-bundles/blob/master/stable/openstack-base/bundle.yaml
.. _openstack-bundles: https://github.com/openstack-charmers/openstack-bundles/
.. _bundle README: https://github.com/openstack-charmers/openstack-bundles/blob/master/stable/openstack-base/README.md
.. _cinder-ceph: https://charmhub.io/cinder-ceph
.. _LP #1892201: https://bugs.launchpad.net/charm-ceph-rbd-mirror/+bug/1892201
