:orphan:

=================================================
Start MySQL InnoDB Cluster from a complete outage
=================================================

Preamble
--------

Regardless of how MySQL InnoDB Cluster services were shut down (gracefully,
hard shutdown, or power outage) a special startup procedure is required in
order to put the cloud database back online.

Procedure
---------

This example will assume that the state of the cloud database is as follows:

.. code-block:: none

   juju status mysql-innodb-cluster

   App                   Version  Status   Scale  Charm                 Store       Channel  Rev  OS      Message
   mysql-innodb-cluster  8.0.25   blocked      3  mysql-innodb-cluster  charmstore  stable     7  ubuntu  Cluster is inaccessible from this instance. Please check logs for details.

   Unit                     Workload  Agent  Machine  Public address  Ports  Message
   mysql-innodb-cluster/0   blocked   idle   0/lxd/2  10.0.0.240             Cluster is inaccessible from this instance. Please check logs for details.
   mysql-innodb-cluster/1   blocked   idle   1/lxd/2  10.0.0.208             Cluster is inaccessible from this instance. Please check logs for details.
   mysql-innodb-cluster/2*  blocked   idle   2/lxd/2  10.0.0.218             Cluster is inaccessible from this instance. Please check logs for details.

Initialise the cluster by running the ``reboot-cluster-from-complete-outage``
action on any mysql-innodb-cluster unit:

.. code-block:: none

   juju run-action --wait mysql-innodb-cluster/1 reboot-cluster-from-complete-outage

.. important::

   If the chosen unit is not the most up-to-date in terms of cluster activity
   the action will fail. However, the action's output messaging will include
   the correct node to use (in terms of its IP address). In such a case, simply
   re-run the action against the proper unit.

The mysql-innodb-cluster application should now be back to a clustered and
healthy state:

.. code-block:: console

   App                   Version  Status  Scale  Charm                 Store       Channel  Rev  OS      Message
   mysql-innodb-cluster  8.0.25   active      3  mysql-innodb-cluster  charmstore  stable     7  ubuntu  Unit is ready: Mode: R/O, Cluster is ONLINE and can tolerate up to ONE failure.

   Unit                     Workload  Agent  Machine  Public address  Ports  Message
   mysql-innodb-cluster/0   active    idle   0/lxd/2  10.0.0.240             Unit is ready: Mode: R/O, Cluster is ONLINE and can tolerate up to ONE failure.
   mysql-innodb-cluster/1   active    idle   1/lxd/2  10.0.0.208             Unit is ready: Mode: R/O, Cluster is ONLINE and can tolerate up to ONE failure.
   mysql-innodb-cluster/2*  active    idle   2/lxd/2  10.0.0.218             Unit is ready: Mode: R/W, Cluster is ONLINE and can tolerate up to ONE failure.

See the :ref:`mysql-innodb-cluster section <mysql_innodb_cluster_startup>` on
the :doc:`Managing power events <../howto/managing-power-events>` page for full
coverage.
