:orphan:

======================
Repair a RabbitMQ node
======================

.. important::

   This page has been identified as being affected by the breaking changes
   introduced between versions 2.9.x and 3.x of the Juju client. Read
   support note :ref:`juju_29_3x_changes` before continuing.

Introduction
~~~~~~~~~~~~

There may be times when a cloud's RabbitMQ cluster experiences difficulties.
When this happens one option to bringing the cluster back to a healthy state is
to repair one of its nodes.

.. important::

   This procedure will not result in cloud downtime providing that there is at
   least one functional RabbitMQ node present.

   Advantageously, it will not trigger the restart of every client application
   (cloud service) that has a relation to the rabbitmq-server application.

Repair vs replacement
^^^^^^^^^^^^^^^^^^^^^

An alternative approach to repairing a node is to replace it. See operation
:doc:`Replace a RabbitMQ node <ops-replace-rabbitmq-node>`.

Below is a comparison of the repair and replacement methods.

+---------+---------------------+---------------------+
| Method  | Pro                 | Con                 |
+=========+=====================+=====================+
| repair  | less time required  | less comprehensive  |
+---------+---------------------+---------------------+
|         | no service restarts |                     |
+---------+---------------------+---------------------+
| replace | comprehensive       | more time required* |
+---------+---------------------+---------------------+
|         |                     | service restarts**  |
+---------+---------------------+---------------------+

\* especially if a new machine must be provisioned

\** can be mitigated with the :doc:`Deferred service events <deferred-events>`
feature

Initial state of the cluster
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

For this example, the :command:`juju status rabbitmq-server` command describes
the initial state of the cluster:

.. code-block:: console

   Model      Controller       Cloud/Region      Version  SLA          Timestamp
   openstack  maas-controller  maas-one/default  2.9.18   unsupported  23:18:34Z

   App              Version  Status  Scale  Charm            Store       Channel  Rev  OS      Message
   rabbitmq-server  3.8.2    active      3  rabbitmq-server  charmstore  stable   118  ubuntu  Unit is ready and clustered

   Unit                Workload  Agent  Machine  Public address  Ports     Message
   rabbitmq-server/0   error     idle   0/lxd/0  10.0.0.177      5672/tcp  ????????????????
   rabbitmq-server/1*  active    idle   1/lxd/0  10.0.0.175      5672/tcp  Unit is ready and clustered
   rabbitmq-server/2   active    idle   2/lxd/0  10.0.0.176      5672/tcp  Unit is ready and clustered

   Machine  State    DNS         Inst id              Series  AZ       Message
   0        started  10.0.0.172  node2                focal   default  Deployed
   0/lxd/0  started  10.0.0.177  juju-64dabb-0-lxd-0  focal   default  Container started
   1        started  10.0.0.173  node4                focal   default  Deployed
   1/lxd/0  started  10.0.0.175  juju-64dabb-1-lxd-0  focal   default  Container started
   2        started  10.0.0.174  node3                focal   default  Deployed
   2/lxd/0  started  10.0.0.176  juju-64dabb-2-lxd-0  focal   default  Container started

In this example the unhealthy node is assumed to be ``rabbitmq-server/0``. This
is the node that is to be repaired.

The status report of the cluster shows that all three nodes are both present
and running. From a healthy node (``rabbitmq-server/2``):

.. code-block:: none

   juju ssh rabbitmq-server/2 sudo rabbitmqctl cluster_status

Partial output:

.. code-block:: console

   Disk Nodes

   rabbit@juju-64dabb-0-lxd-0
   rabbit@juju-64dabb-1-lxd-0
   rabbit@juju-64dabb-2-lxd-0

   Running Nodes

   rabbit@juju-64dabb-0-lxd-0
   rabbit@juju-64dabb-1-lxd-0
   rabbit@juju-64dabb-2-lxd-0

Pause the unhealthy node's service
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Pause the RabbitMQ service on the unhealthy node/unit:

.. code-block:: none

   juju run rabbitmq-server/0 pause

Identify the unhealthy node's hostname
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The status report of the cluster should show that a node is no longer running.

From a healthy node:

.. code-block:: none

   juju ssh rabbitmq-server/2 sudo rabbitmqctl cluster_status

The cluster's status output now includes:

.. code-block:: console

   Disk Nodes

   rabbit@juju-64dabb-0-lxd-0
   rabbit@juju-64dabb-1-lxd-0
   rabbit@juju-64dabb-2-lxd-0

   Running Nodes

   rabbit@juju-64dabb-1-lxd-0
   rabbit@juju-64dabb-2-lxd-0

The hostname of the unhealthy node is the one that is no longer running. In
this example, it is **rabbit@juju-64dabb-0-lxd-0**.

Remove the unhealthy node from the cluster
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Apply the ``forget-cluster-node`` action to a healthy node's unit and refer to
the unhealthy node by its now-known hostname. This removes the unhealthy node
from the cluster:

.. code-block:: none

   juju run rabbitmq-server/2 forget-cluster-node node=rabbit@juju-64dabb-0-lxd-0

The cluster's status output should now include:

.. code-block:: console

   Disk Nodes

   rabbit@juju-64dabb-1-lxd-0
   rabbit@juju-64dabb-2-lxd-0

   Running Nodes

   rabbit@juju-64dabb-1-lxd-0
   rabbit@juju-64dabb-2-lxd-0

Refresh the unhealthy node and rejoin it to the cluster
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

We refresh the unhealthy node and rejoin it to the cluster by invoking commands
directly on the machine that is hosting the unhealthy node. Begin by connecting
to it:

.. code-block:: none

   juju ssh rabbitmq-server/0

#. Move the current database out of the way:

   .. note::

      Normally, a previously working node cannot join a cluster with its old
      database intact. It must first be removed or renamed.

   .. code-block:: none

      > sudo mv /var/lib/rabbitmq/{mnesia,mnesia.bak}

#. Start the RabbitMQ service in standalone (unclustered) mode

   .. code-block:: none

      > sudo rabbitmq-server -detached

#. Stop the RabbitMQ server but keep the Erlang VM running

   .. code-block:: none

      > sudo rabbitmqctl stop_app

#. Rejoin the unhealthy node to the cluster

   Rejoin the unhealthy node to the cluster by referencing an existing cluster
   member:

   .. code-block:: none

      > sudo rabbitmqctl join_cluster rabbit@juju-64dabb-2-lxd-0

   The cluster's status output should now include:

   .. code-block:: console

      Disk Nodes

      rabbit@juju-64dabb-0-lxd-0
      rabbit@juju-64dabb-1-lxd-0
      rabbit@juju-64dabb-2-lxd-0

      Running Nodes

      rabbit@juju-64dabb-0-lxd-0
      rabbit@juju-64dabb-2-lxd-0

#. Start a refreshed RabbitMQ server

   .. code-block:: none

      > sudo rabbitmqctl start_app

   The cluster's status output should now include:

   .. code-block:: console

      Disk Nodes

      rabbit@juju-64dabb-0-lxd-0
      rabbit@juju-64dabb-1-lxd-0
      rabbit@juju-64dabb-2-lxd-0

      Running Nodes

      rabbit@juju-64dabb-0-lxd-0
      rabbit@juju-64dabb-1-lxd-0
      rabbit@juju-64dabb-2-lxd-0

   The repaired node has rejoined the cluster and is now running.

#. Stop the RabbitMQ service

   The RabbitMQ service must be stopped in the current environment in order for
   it to be managed by Juju:

   .. code-block:: none

      > sudo rabbitmqctl stop

Resume the repaired node's service
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Resume the RabbitMQ service on the repaired node/unit:

.. code-block:: none

   juju run rabbitmq-server/0 resume

Verify model health
~~~~~~~~~~~~~~~~~~~

Verify the model's health with the :command:`juju status rabbitmq-server`
command. Its partial output should look like:

.. code-block:: console

   App              Version  Status  Scale  Charm            Store       Channel  Rev  OS      Message
   rabbitmq-server  3.8.2    active      3  rabbitmq-server  charmstore  stable   118  ubuntu  Unit is ready and clustered

   Unit                Workload  Agent  Machine  Public address  Ports     Message
   rabbitmq-server/0   active    idle   0/lxd/0  10.0.0.177      5672/tcp  Unit is ready and clustered
   rabbitmq-server/1*  active    idle   1/lxd/0  10.0.0.175      5672/tcp  Unit is ready and clustered
   rabbitmq-server/2   active    idle   2/lxd/0  10.0.0.176      5672/tcp  Unit is ready and clustered
