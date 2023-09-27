:orphan:

==================================
Remove a MySQL InnoDB Cluster node
==================================

.. important::

   This page has been identified as being affected by the breaking changes
   introduced between versions 2.9.x and 3.x of the Juju client. Read
   support note :ref:`juju_29_3x_changes` before continuing.

Preamble
--------


.. caution::

   MySQL InnoDB Cluster requires a minimum of three instances. Do not attempt
   to remove a node unless there are at least four nodes present.

Procedure
---------

Check the current state of the cloud, scale out by adding a single mysql-innodb-cluster
node, and verify that the new node is working.

Current state
~~~~~~~~~~~~~

Gather basic information about the current state of the cloud in terms of the
mysql-innodb-cluster application application:

Get the Juju status of the application:

.. code-block:: none

   juju status mysql-innodb-cluster

   Model  Controller           Cloud/Region             Version  SLA          Timestamp
   mysql  tinwood-serverstack  serverstack/serverstack  2.9.37   unsupported  14:41:21Z

   App                   Version  Status  Scale  Charm                 Channel  Rev  Exposed  Message
   mysql-innodb-cluster  8.0.33   active    2/3  mysql-innodb-cluster             0  no       Unit is ready: Mode: R/W, Cluster is NOT tolerant to any failures. 1 member is not active.

   Unit                     Workload  Agent  Machine  Public address  Ports  Message
   mysql-innodb-cluster/0*  active    idle   3        172.20.0.250           Unit is ready: Mode: R/W, Cluster is NOT tolerant to any failures. 1 member is not active.
   mysql-innodb-cluster/1   active    idle   4        172.20.0.172           Unit is ready: Mode: R/O, Cluster is NOT tolerant to any failures. 1 member is not active.
   mysql-innodb-cluster/2   unknown   lost   5        172.20.0.36            agent lost, see 'juju show-status-log mysql-innodb-cluster/2'

   Machine  State    Address       Inst id                               Series  AZ    Message
   3        started  172.20.0.250  621c3f6d-e64f-4d8b-af13-acd6cb6fe059  jammy   nova  ACTIVE
   4        started  172.20.0.172  965a2a05-51cf-4730-95d6-d881ed1eae61  jammy   nova  ACTIVE
   5        down     172.20.0.36   3d2a20e9-cc80-48ab-b29f-8dae6781dddd  jammy   nova  ACTIVE

Check cluster status:

.. code-block:: none

   juju run mysql-innodb-cluster/leader cluster-status

   unit-mysql-innodb-cluster-0:
     UnitId: mysql-innodb-cluster/0
     id: "28"
     results:
       cluster-status: '{"clusterName": "jujuCluster", "defaultReplicaSet": {"name":
         "default", "primary": "172.20.0.250:3306", "ssl": "REQUIRED", "status": "OK_NO_TOLERANCE",
         "statusText": "Cluster is NOT tolerant to any failures. 1 member is not active.",
         "topology": {"172.20.0.172:3306": {"address": "172.20.0.172:3306", "mode": "R/O",
         "readReplicas": {}, "replicationLag": null, "role": "HA", "status": "ONLINE",
         "version": "8.0.33"}, "172.20.0.250:3306": {"address": "172.20.0.250:3306",
         "mode": "R/W", "readReplicas": {}, "replicationLag": null, "role": "HA", "status":
         "ONLINE", "version": "8.0.33"}, "172.20.0.36:3306": {"address": "172.20.0.36:3306",
         "mode": "n/a", "readReplicas": {}, "role": "HA", "shellConnectError": "MySQL
         Error 2003 (HY000): Can''t connect to MySQL server on ''172.20.0.36'' (113)",
         "status": "(MISSING)"}}, "topologyMode": "Single-Primary"}, "groupInformationSourceMember":
         "172.20.0.250:3306"}'
     status: completed
     timing:
       completed: 2023-06-01 14:42:42 +0000 UTC
       enqueued: 2023-06-01 14:42:36 +0000 UTC
       started: 2023-06-01 14:42:36 +0000 UTC

The cluster status tidied up looks like:

.. code-block:: json

   {
      "clusterName":"jujuCluster",
      "defaultReplicaSet":{
         "name":"default",
         "primary":"172.20.0.250:3306",
         "ssl":"REQUIRED",
         "status":"OK_NO_TOLERANCE",
         "statusText":"Cluster is NOT tolerant to any failures. 1 member is not active.",
         "topology":{
            "172.20.0.172:3306":{
               "address":"172.20.0.172:3306",
               "mode":"R/O",
               "readReplicas":{
               },
               "replicationLag":null,
               "role":"HA",
               "status":"ONLINE",
               "version":"8.0.33"
            },
            "172.20.0.250:3306":{
               "address":"172.20.0.250:3306",
               "mode":"R/W",
               "readReplicas":{
               },
               "replicationLag":null,
               "role":"HA",
               "status":"ONLINE",
               "version":"8.0.33"
            },
            "172.20.0.36:3306":{
               "address":"172.20.0.36:3306",
               "mode":"n/a",
               "readReplicas":{
               },
               "role":"HA",
               "shellConnectError":"MySQL
         Error 2003 (HY000): Can''t connect to MySQL server on ''172.20.0.36'' (113)",
               "status":"(MISSING)"
            }
         },
         "topologyMode":"Single-Primary"
      },
      "groupInformationSourceMember":"172.20.0.250:3306"
   }

i.e. the ``172.20.0.36`` unit is missing.

If there are only 3 instances, then add a new instance:

.. code-block:: none

   juju add-unit mysql-innodb-cluster

The above command may need altering with additional options such as
constraining where the unit should be placed and memory constraints. Please
explore the existing units to discover the constraints.

Finally, ``juju status mysql-innodb-cluster`` will show something like:

.. code-block:: none

   mysql  tinwood-serverstack  serverstack/serverstack  2.9.37   unsupported  15:10:46Z

   App                   Version  Status  Scale  Charm                 Channel  Rev  Exposed  Message
   mysql-innodb-cluster  8.0.33   active    3/4  mysql-innodb-cluster             0  no       Unit is ready: Mode: R/W, Cluster is ONLINE and can tolerate up to ONE failure. 1 m
   ember is not active.

   Unit                     Workload  Agent  Machine  Public address  Ports  Message
   mysql-innodb-cluster/0*  active    idle   3        172.20.0.250           Unit is ready: Mode: R/W, Cluster is ONLINE and can tolerate up to ONE failure. 1 member is not acti
   ve.
   mysql-innodb-cluster/1   active    idle   4        172.20.0.172           Unit is ready: Mode: R/O, Cluster is ONLINE and can tolerate up to ONE failure. 1 member is not acti
   ve.
   mysql-innodb-cluster/2   unknown   lost   5        172.20.0.36            agent lost, see 'juju show-status-log mysql-innodb-cluster/2'
   mysql-innodb-cluster/3   active    idle   15       172.20.0.139           Unit is ready: Mode: R/O, Cluster is ONLINE and can tolerate up to ONE failure. 1 member is not acti
   ve.

   Machine  State    Address       Inst id                               Series  AZ    Message
   3        started  172.20.0.250  621c3f6d-e64f-4d8b-af13-acd6cb6fe059  jammy   nova  ACTIVE
   4        started  172.20.0.172  965a2a05-51cf-4730-95d6-d881ed1eae61  jammy   nova  ACTIVE
   5        down     172.20.0.36   3d2a20e9-cc80-48ab-b29f-8dae6781dddd  jammy   nova  ACTIVE
   15       started  172.20.0.139  5017e355-3df7-408b-b693-eb2726d2fa43  jammy   nova  ACTIVE


There are now 4 units, one is dead, and the other 3 have re-formed a successful cluster.

Remove the database instance from the cluster
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Use the ``remove-instance`` action to remove the database instance from the
cluster. The action must be run on a functioning member of the cluster, which
may not necessarily be the leader. Use the ``juju status`` output above to
determine a working member. In the examples below, ``2`` is used as the working
member of the cluster.

While the instance is running:

.. code-block:: none

   juju run mysql-innodb-cluster/2 remove-instance address=<instance-ip-address>

Use the force argument if the host is down (or no longer exists):

.. code-block:: none

   juju run mysql-innodb-cluster/2 remove-instance address=<instance-ip-address> force=True

.. warning::

   Removing the instance from the cluster is particularly important when the
   addition of a subsequent node re-uses the original IP address.

Check cluster status:

.. code-block:: none

   juju run mysql-innodb-cluster/2 cluster-status

   {
      "clusterName":"jujuCluster",
      "defaultReplicaSet":{
         "name":"default",
         "primary":"172.20.0.250:3306",
         "ssl":"REQUIRED",
         "status":"OK",
         "statusText":"Cluster is ONLINE and can tolerate up to ONE failure.",
         "topology":{
            "172.20.0.139:3306":{
               "address":"172.20.0.139:3306",
               "mode":"R/O",
               "readReplicas":{
               },
               "replicationLag":null,
               "role":"HA",
               "status":"ONLINE",
               "version":"8.0.33"
            },
            "172.20.0.172:3306":{
               "address":"172.20.0.172:3306",
               "mode":"R/O",
               "readReplicas":{
               },
               "replicationLag":null,
               "role":"HA",
               "status":"ONLINE",
               "version":"8.0.33"
            },
            "172.20.0.250:3306":{
               "address":"172.20.0.250:3306",
               "mode":"R/W",
               "readReplicas":{
               },
               "replicationLag":null,
               "role":"HA",
               "status":"ONLINE",
               "version":"8.0.33"
            }
         },
         "topologyMode":"Single-Primary"
      },
      "groupInformationSourceMember":"172.20.0.250:3306"
   }"'"

Scale back
~~~~~~~~~~

Remove the database unit from the model.  Depending on the state of the unit,
it may be necessary to use a ``remove-machine`` using the ``--force`` option.

.. code-block:: none

   juju remove-unit mysql-innodb-cluster/2
   juju remove-machine 5 --force

The status output should eventually look similar to:

.. code-block:: console

   Model      Controller  Cloud/Region      Version  SLA          Timestamp
   mysql  tinwood-serverstack  serverstack/serverstack  2.9.37   unsupported  11:53:03Z

   App                   Version  Status  Scale  Charm                 Channel  Rev  Exposed  Message
   mysql-innodb-cluster  8.0.33   active      3  mysql-innodb-cluster             3  no       Unit is ready: Mode: R/O, Cluster is ONLINE and can tolerate up to ONE failure.

   Unit                     Workload  Agent  Machine  Public address  Ports  Message
   mysql-innodb-cluster/0*  active    idle   3        172.20.0.250           Unit is ready: Mode: R/O, Cluster is ONLINE and can tolerate up to ONE failure.
   mysql-innodb-cluster/1   active    idle   4        172.20.0.172           Unit is ready: Mode: R/O, Cluster is ONLINE and can tolerate up to ONE failure.
   mysql-innodb-cluster/3   active    idle   15       172.20.0.139           Unit is ready: Mode: R/W, Cluster is ONLINE and can tolerate up to ONE failure.


Verification
~~~~~~~~~~~~

Verify that the mysql cluster is healthy, and that the service applications are
functioning correctly:

.. code-block:: none

   juju run mysql-innodb-cluster/leader cluster-status

And run an openstack command:

.. code-block:: none

   openstack endpoint list
