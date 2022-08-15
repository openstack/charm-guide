==============================================
ceph charm: migration to ceph-mon and ceph-osd
==============================================

.. note::

   This page describes a procedure that may be required when performing an
   upgrade of an OpenStack cloud. Please read the more general
   :doc:`../../admin/upgrades/overview` before attempting any of the
   instructions given here.

In order to continue to receive updates to newer Ceph versions, and for general
improvements and features in the charms to deploy Ceph, users of the ceph charm
should migrate existing services to using ceph-mon and ceph-osd.

.. note::

   This example migration assumes that the ceph charm is deployed to machines
   0, 1 and 2 with the ceph-osd charm deployed to other machines within the
   model.

Procedure
---------

Upgrade charms
~~~~~~~~~~~~~~

The entire suite of charms used to manage the cloud should be upgraded to the
latest stable charm revision before any major change is made to the cloud such
as the current migration to these new charms. See `Charms upgrade`_ for
guidance.

Deploy ceph-mon
~~~~~~~~~~~~~~~

.. warning::

   Every new ceph-mon unit introduced will result in a Ceph monitor receiving a
   new IP address. However, due to an issue in Nova, this fact is not
   propagated completely throughout the cloud under certain circumstances,
   thereby affecting Ceph RBD volume reachability.

   Any instances previously deployed using Cinder to interface with Ceph, or
   using Nova's ``libvirt-image-backend=rbd`` setting will require a manual
   database update to change to the new addresses. For Cinder, its stale data
   will also need to be updated in the 'block_device_mapping' table.

   Failure to do this can result in instances being unable to start as their
   volumes cannot be reached. See bug `LP #1452641`_.

First deploy the ceph-mon charm; if the existing ceph charm is deployed to machines
0, 1 and 2, you can place the ceph-mon units in LXD containers on these machines:

.. code-block:: none

   juju deploy --to lxd:0 ceph-mon
   juju config ceph-mon no-bootstrap=True
   juju add-unit --to lxd:1 ceph-mon
   juju add-unit --to lxd:2 ceph-mon

These units will install ceph, but will not bootstrap into a running monitor cluster.

Bootstrap ceph-mon from ceph
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Next, we'll use the existing ceph application to bootstrap the new ceph-mon units:

.. code-block:: none

   juju add-relation ceph ceph-mon

Once this process has completed, you should have a Ceph MON cluster of 6 units;
this can be verified on any of the ceph or ceph-mon units:

.. code-block:: none

   sudo ceph -s

Deploy ceph-osd to ceph units
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In order to retain any running Ceph OSD processes on the ceph units, the ceph-osd
charm must be deployed to the existing machines running the ceph units:

.. code-block:: none

   juju config ceph-osd osd-reformat=False
   juju add-unit --to 0 ceph-osd
   juju add-unit --to 1 ceph-osd
   juju add-unit --to 2 ceph-osd

As of the 18.05 charm release, the ``osd-reformat`` configuration option has
been completely removed.

The charm installation and configuration will not impact any existing running
Ceph OSDs.

Relate ceph-mon to all ceph clients
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The new ceph-mon units now need to be related to the ceph-osd application:

.. code-block:: none

   juju add-relation ceph-mon ceph-osd

Depending on your deployment you'll also need to add relations for other
applications, for example:

.. code-block:: none

   juju add-relation ceph-mon cinder-ceph
   juju add-relation ceph-mon glance
   juju add-relation ceph-mon nova-compute
   juju add-relation ceph-mon ceph-radosgw
   juju add-relation ceph-mon gnocchi

Once hook execution completes across all units, each client should be
configured with six MON addresses.

Remove the ceph application
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Its now safe to remove the ceph application from your deployment:

.. code-block:: none

   juju remove-application ceph

As each unit of the ceph application is destroyed, its stop hook will remove
the MON process from the Ceph cluster monmap and disable Ceph MON and MGR
processes running on the machine; any Ceph OSD processes remain untouched and
are now owned by the ceph-osd units deployed alongside ceph.

.. LINKS
.. _Charms upgrade: upgrade-charms.html
.. _LP #1452641: https://bugs.launchpad.net/nova/+bug/1452641
