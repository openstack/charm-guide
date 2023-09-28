========================================
Ceph RADOS Gateway multisite replication
========================================

Overview
++++++++

Ceph RADOS Gateway (RGW) native replication between ceph-radosgw applications
is supported both within a single model and between different models. By
default, each application will accept write operations.

.. note::

   Multisite replication is supported starting with Ceph Luminous.

.. warning::

   Converting from a standalone deployment to a replicated deployment is not
   supported.

Deployment
++++++++++

.. note::

    Example bundles for the us-west and us-east models can be found
    in the `bundles` subdirectory of the ceph-radosgw charm.

To deploy the ceph-radosgw charm in this configuration ensure that the
following configuration options are set on the instances of the ceph-radosgw
deployed - in this example `rgw-us-east` and `rgw-us-west` are both instances
of the ceph-radosgw charm:

.. code::

    rgw-us-east:
      realm: replicated
      zonegroup: us
      zone: us-east
    rgw-us-west:
      realm: replicated
      zonegroup: us
      zone: us-west

.. note::

    The realm and zonegroup configuration must be identical between instances
    of the ceph-radosgw application participating in the multi-site
    deployment; the zone configuration must be unique per application.

When deploying with this configuration the ceph-radosgw applications will
deploy into a blocked state until the primary/secondary (cross-model) relation
is added.

Typically each ceph-radosgw deployment will be associated with a separate
ceph cluster at different physical locations - in this example the deployments
are in different models ('us-east' and 'us-west').

One ceph-radosgw application acts as the initial primary for the deployment -
setup the primary relation endpoint as the provider of the offer for the
cross-model relation:

.. code::

    juju offer -m us-east rgw-us-east:primary

The cross-model relation offer can then be consumed in the other model and
related to the secondary ceph-radosgw application:

.. code::

    juju consume -m us-west admin/us-east.rgw-us-east
    juju add-relation -m us-west rgw-us-west:secondary rgw-us-east:primary

Once the relation has been added the realm, zonegroup and zone configuration
will be created in the primary deployment and then synced to the secondary
deployment.

The current sync status can be validated from either model:

.. code::

    juju ssh -m us-east ceph-mon/0
    sudo radosgw-admin sync status
              realm 142eb39c-67c4-42b3-9116-1f4ffca23964 (replicated)
          zonegroup 7b69f059-425b-44f5-8a21-ade63c2034bd (us)
               zone 4ee3bc39-b526-4ac9-a233-64ebeacc4574 (us-east)
      metadata sync no sync (zone is master)
          data sync source: db876cf0-62a8-4b95-88f4-d0f543136a07 (us-west)
                            syncing
                            full sync: 0/128 shards
                            incremental sync: 128/128 shards
                            data is caught up with source

Once the deployment is complete, the default zone and zonegroup can
optionally be tidied using the 'tidydefaults' action:

.. code::

    juju run-action -m us-west --wait rgw-us-west/0 tidydefaults

.. warning::

    This operation is not reversible.

Failover/Recovery
+++++++++++++++++

In the event that the site hosting the zone which is the primary for metadata
(in this example us-east) has an outage, the primary metadata zone must be
failed over to the secondary site; this operation is performed using the 'promote'
action:

.. code::

    juju run-action -m us-west --wait rgw-us-west/0 promote

Once this action has completed, the secondary site will be the primary for metadata
updates and the deployment will accept new uploads of data.

Once the failed site has been recovered it will resync and resume as a secondary
to the promoted primary site (us-west in this example).

The primary metadata zone can be failed back to its original location once resync
has completed using the 'promote' action:

.. code::

    juju run-action -m us-east --wait rgw-us-east/0 promote

Read/write vs Read-only
-----------------------

By default all zones within a deployment will be read/write capable but only
the primary zone can be used to create new containers.

Non-primary zones can optionally be marked as read-only by using the 'readonly'
action:

.. code::

    juju run-action -m us-east --wait rgw-us-east/0 readonly

a zone that is currently read-only can be switched to read/write mode by either
promoting it to be the current primary or by using the 'readwrite' action:

.. code::

    juju run-action -m us-east --wait rgw-us-east/0 readwrite

