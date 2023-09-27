:orphan:

=======================================
Scale back the ovn-central application
=======================================

.. important::

   This page has been identified as being affected by the breaking changes
   introduced between versions 2.9.x and 3.x of the Juju client. Read
   support note :ref:`juju_29_3x_changes` before continuing.

Preamble
--------

Clean downscaling of the ovn-central application is supported from release
23.03 onwards. Earlier versions of the charm will require some manual steps.

Think about the impact
----------------------

OVN central is using the Raft consensus algorithm to facilitate HA. Raft

* Tolerates up to (N-1)/2 node failures
* Requires minimum quorum of (N/2)+1 members

Changes to the number of members in the OVN cluster affect its fault tolerance
as well as its minimum requirements for quorum. Before you downscale your
cluster, think about the impact it will have on both of these properties.

It is not recommended to downscale ovn-central application below 3 members.


Procedure for releases before 23.03
-----------------------------------

With older releases of ovn-central charm, the operator can run
``juju remove-unit`` command, but internally, OVN cluster will not perform
reconfiguration and it will keep expecting servers from the removed unit to
rejoin the cluster. To cleanly remove units, you have to complete a few manual
steps.

Log into the unit that you intend to remove (using ``juju ssh``) and execute
the following commands as root:

.. code-block:: none

   ovn-appctl -t /var/run/ovn/ovnsb_db.ctl cluster/leave OVN_Southbound
   ovn-appctl -t /var/run/ovn/ovnnb_db.ctl cluster/leave OVN_Northbound

This will cause OVN servers hosted on this unit to gracefully leave both
Southbound and Northbound OVN clusters.

Perform unit removal with:

.. code-block:: none

   juju remove-unit <UNIT_NAME>

To verify that the downscaling completed successfully, log into one of the
remaining units of ovn-central and check the state of both clusters (again as
root).

.. code-block:: none

   ovn-appctl -t /var/run/ovn/ovnsb_db.ctl cluster/status OVN_Southbound
   ovn-appctl -t /var/run/ovn/ovnnb_db.ctl cluster/status OVN_Northbound

If both clusters have an expected number of members, you are done. However if
any of the clusters did not perform reconfiguration and removed servers are
still hanging around, you can kick them manually using following command where
CLUSTER_NAME is either "OVN_Southbound" or "OVN_Northbound" and SERVER_ID is
a short hexadecimal number from "cluster/status" output.

.. code-block:: none

   ovn-appctl -t /var/run/ovn/ovnsb_db.ctl cluster/kick <CLUSTER_NAME> <SERVER_ID>

.. note::

   The ``cluster/kick`` command does everything needed to decrease the number
   of cluster members by one. It both removes the targeted server and informs
   the remaining members so their view of the cluster can be automatically
   updated.

Procedure for release 23.03 and after
-------------------------------------

Starting with this release, the removed ovn-central unit will attempt to perform
a graceful departure from the cluster so the operator should not need to do
anything else than remove the unit with:

.. code-block:: none

   juju remove-unit <UNIT_NAME>

To verify that the unit departed cluster cleanly, wait for the ovn-central
application to settle and run:

.. code-block:: none

   juju run <OVN_CENTRAL_UNIT> cluster-status

This output will show yaml-formatted status of both Southbound and Northbound
OVN clusters. Each cluster status will contain key "unit_map", if this list
does not contain any servers in category "UNKNOWN", it means that downscaling
completed successfully.

Example of "unit_map" after successful downscaling:

.. code-block:: console

   unit_map:
    ovn-central/3: 7ed2
    ovn-central/1: f1ca
    ovn-central/2: 92d5


However if there are "UNKNOWN" servers, for example like this:

.. code-block:: console

      unit_map:
        ovn-central/3: 7ed2
        ovn-central/1: f1ca
        ovn-central/2: 92d5
        UNKNOWN:
        - ba21

It means that downscaling did not complete successfully, and you'll have to
manually kick servers listed as "UNKNOWN" using the `cluster-kick`_ action
provided by the charm.

.. LINKS
.. _cluster-kick: https://charmhub.io/ovn-central/actions?channel=latest/edge#cluster-kick
