=========================
Upgrade: 2023.1 to 2023.2
=========================

This page contains notes specific to the 2023.1 to 2023.2 upgrade path. See
the main :doc:`../../admin/upgrades/openstack` page for full coverage.

.. _az_option_removal:

Nova ``AvailabilityZoneFilter`` removal in 2023.2
-------------------------------------------------

The AvailabilityZoneFilter option was removed from Nova in 2023.2 Bobcat (see
the `Nova 2023.2 upgrade notes`_).

In order for the scheduler to honour an availability zone request, there must
now be a Placement aggregate that matches the Nova host aggregate that was
assigned to an availability zone.

Since Nova 18.0.0 (OpenStack Rocky), the :command:`nova-api` service attempts
to automatically mirror the association of a compute host with a Placement
aggregate when the host is added/removed to/from a Nova host aggregate.

The below commands will assist in confirming whether this mirroring has
occurred:

1. List Nova aggregates:

.. code-block:: none

   openstack aggregate list
   +----+------+-------------------+
   | ID | Name | Availability Zone |
   +----+------+-------------------+
   |  1 | myag | myaz              |
   +----+------+-------------------+

2. Show details on one such Nova aggregate (shows compute host name and Nova
   aggregate UUID):

.. code-block:: none

   openstack aggregate show --column availability_zone --column hosts --column uuid myag
   +-------------------+------------------------------------------------------+
   | Field             | Value                                                |
   +-------------------+------------------------------------------------------+
   | availability_zone | myaz                                                 |
   | hosts             | juju-2c7db9-zaza-2349f0f509d3-14.project.serverstack |
   | uuid              | 4dd789b7-b4c3-45f1-8b2b-a6f5a8c37d55                 |
   +-------------------+------------------------------------------------------+

3. List resource providers (shows compute host UUIDs as known to Placement):

.. code-block:: none

   openstack resource provider list --column uuid --column name
   +--------------------------------------+------------------------------------------------------+
   | uuid                                 | name                                                 |
   +--------------------------------------+------------------------------------------------------+
   | 482399c5-9ed7-4d4d-bdcf-c076dae99f2d | juju-2c7db9-zaza-2349f0f509d3-14.project.serverstack |
   | d1322831-94db-4628-9adc-3406014d24e4 | juju-2c7db9-zaza-2349f0f509d3-15.project.serverstack |
   | 624c0f64-8a2b-47c7-9ea6-e3f1de611bc2 | juju-2c7db9-zaza-2349f0f509d3-16.project.serverstack |
   +--------------------------------------+------------------------------------------------------+

4. List Placement aggregates for a compute host that is now known to be
   associated with a Nova aggregate:

.. code-block:: none

   openstack resource provider aggregate list --column uuid 482399c5-9ed7-4d4d-bdcf-c076dae99f2d
   +--------------------------------------+
   | uuid                                 |
   +--------------------------------------+
   | 4dd789b7-b4c3-45f1-8b2b-a6f5a8c37d55 |
   +--------------------------------------+

The UUID shown should be the same as that displayed in the output in step #2.

Manual intervention will be required if an AZ-assigned Nova aggregate is not
also associated with a Placement aggregate. This is done with the
:command:`openstack resource provider aggregate set` command (see `Aggregates
in Placement`_) in the Nova documentation.

.. _bluestore_migration:

FileStore support removal in Ceph Reef
--------------------------------------

Charmed OpenStack 2023.2 ships with Ceph Reef, which has `removed support for
the OSD FileStore format`_, leaving support for only the BlueStore format.

.. warning::

   Data loss may occur if you attempt to upgrade to Ceph Reef when FileStore
   OSDs are present. As an extra precaution, the ceph-osd charm will attempt to
   detect FileStore OSDs, and, if any are found, it will block a payload
   upgrade.

Before upgrading the payload ("OpenStack upgrade") of any of the Ceph charms,
migrate all FileStore OSDs to BlueStore by following the upstream
documentation: `BlueStore migration strategies`_.

.. LINKS
.. _Nova 2023.2 upgrade notes: https://docs.openstack.org/releasenotes/nova/2023.2.html#upgrade-notes
.. _Aggregates in Placement: https://docs.openstack.org/nova/latest/admin/aggregates.html#aggregates-in-placement
.. _removed support for the OSD FileStore format: https://docs.ceph.com/en/latest/rados/configuration/storage-devices/#filestore
.. _BlueStore migration strategies: https://docs.ceph.com/en/quincy/rados/operations/bluestore-migration/
