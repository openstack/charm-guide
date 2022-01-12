:orphan:

======================================
Restart a partitioned RabbitMQ cluster
======================================

Introduction
~~~~~~~~~~~~

The RabbitMQ cluster can become `partitioned`_, leading to a split-brain
scenario, and can be caused by factors such as network instability, message
queue load, or RabbitMQ host restarts. Partitions can cause significant
disruption to a cloud, and intervention is required if the cluster cannot
recover on its own. This involves stopping and starting the RabbitMQ nodes in a
controlled manner, which should not result in lost data (queue and user
permissions).

.. important::

   The default RabbitMQ partition handling strategy in Charmed OpenStack is
   ``ignore`` and is generally not considered a production-grade setting. Use
   the ``cluster-partition-handling`` rabbitmq-server charm option to change
   it.

Initial state of the cluster
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

For this example, the :command:`juju status rabbitmq-server` command describes
the initial state of the cluster:

.. code-block:: console

   App              Version  Status  Scale  Charm            Store       Channel  Rev  OS      Message
   rabbitmq-server  3.8.2    active      3  rabbitmq-server  charmstore  stable   117  ubuntu  Unit is ready and clustered

   Unit                Workload  Agent  Machine  Public address  Ports     Message
   rabbitmq-server/1*  active    idle   1/lxd/5  10.246.114.55   5672/tcp  Unit is ready and clustered
   rabbitmq-server/2   active    idle   0/lxd/5  10.246.114.56   5672/tcp  Unit is ready and clustered
   rabbitmq-server/3   active    idle   2/lxd/7  10.246.114.59   5672/tcp  Unit is ready and clustered

Each cluster node can report on its view of the cluster's status. To check what
the node associated with unit ``rabbitmq-server/1`` sees:

.. code-block:: none

   juju ssh rabbitmq-server/1 sudo rabbitmqctl cluster_status

Here are the more pertinent sections of the status report that each node should
report while the cluster is in a healthy state:

.. code-block:: none

   Disk Nodes

   rabbit@juju-4d3a31-0-lxd-5
   rabbit@juju-4d3a31-1-lxd-5
   rabbit@juju-4d3a31-2-lxd-7

   Running Nodes

   rabbit@juju-4d3a31-0-lxd-5
   rabbit@juju-4d3a31-1-lxd-5
   rabbit@juju-4d3a31-2-lxd-7

   Network Partitions

   (none)

Loss of connectivity for one node
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In this example, it is then assumed that the RabbitMQ node represented by unit
``rabbitmq-server/1`` loses contact with its cluster members.

First re-establish the lost node's network connectivity network.

When partitioning is detected by a node, its cluster status will usually show
that the number of running nodes has decreased, and that the partitions section
may contain extra information. For example:

.. code-block:: console

   Disk Nodes

   rabbit@juju-4d3a31-0-lxd-5
   rabbit@juju-4d3a31-1-lxd-5
   rabbit@juju-4d3a31-2-lxd-7

   Running Nodes

   rabbit@juju-4d3a31-0-lxd-5
   rabbit@juju-4d3a31-2-lxd-7

   Network Partitions

   Node rabbit@juju-4d3a31-0-lxd-5 cannot communicate with rabbit@juju-4d3a31-1-lxd-5
   Node rabbit@juju-4d3a31-2-lxd-7 cannot communicate with rabbit@juju-4d3a31-1-lxd-5

.. important::

   Querying one of the remaining nodes for cluster status is not a definitive
   check for the occurrence of a network partition. Often partitioning will be
   detected only once network connectivity is restored for the downed node or
   partitioning may only be seen from the viewpoint of the downed node.

Determine node health
~~~~~~~~~~~~~~~~~~~~~

Identify the unhealthy nodes and the healthy nodes.

In this example, the node represented by unit ``rabbitmq-server/1`` fell off
the network. This would be an unhealthy node (a "bad" node).

The remaining nodes can be considered as equally healthy if there is no
particular reason to consider them otherwise. In this example, we'll consider
the node represented by unit ``rabbitmq-server/2`` to be the most healthy node
(the "good" node).

Leave the good node running
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Stop all the nodes, starting with the bad node, with the exception of the
designated good node. This will result in a single-node cluster.

Here, we will stop these nodes (in the order given):

#. ``rabbitmq-server/1``
#. ``rabbitmq-server/3``

It is recommended to check the status (by querying the good node) after each
stoppage to ensure that the node as actually stopped:

.. code-block:: none

   juju ssh rabbitmq-server/1 sudo rabbitmqctl stop_app
   juju ssh rabbitmq-server/2 sudo rabbitmqctl cluster_status
   juju ssh rabbitmq-server/3 sudo rabbitmqctl stop_app
   juju ssh rabbitmq-server/2 sudo rabbitmqctl cluster_status

The final status should show that there is a sole running node (the good node)
and no network partitions:

.. code-block:: console

   Disk Nodes

   rabbit@juju-4d3a31-0-lxd-5
   rabbit@juju-4d3a31-1-lxd-5
   rabbit@juju-4d3a31-2-lxd-7

   Running Nodes

   rabbit@juju-4d3a31-0-lxd-5

   Network Partitions

   (none)

Bring up a single-node cluster
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The single-node cluster should come up on its own.

If there are hook errors visible in the :command:`juju status` output attempt
to resolve them. For instance, if the ``rabbitmq-server/1`` unit shows ``hook
failed: "update-status"`` then:

.. code-block:: none

   juju resolve --no-retry rabbitmq-server/1

It's important to let the single-node cluster settle before continuing. Wait
until workload status messages cease to change. This can take at least five
minutes. Resolve any hook errors that may appear.

One test of cluster health is to repetitively query the list of Compute
services. An updating timestamp is indicative of a normal flow of messages
through RabbitMQ:

.. code-block:: none

   openstack compute service list

   +----+----------------+---------------------+----------+---------+-------+----------------------------+
   | ID | Binary         | Host                | Zone     | Status  | State | Updated At                 |
   +----+----------------+---------------------+----------+---------+-------+----------------------------+
   |  1 | nova-scheduler | juju-1531de-0-lxd-3 | internal | enabled | up    | 2022-01-14T00:27:39.000000 |
   |  5 | nova-conductor | juju-1531de-0-lxd-3 | internal | enabled | up    | 2022-01-14T00:27:30.000000 |
   | 13 | nova-compute   | node-fontana.maas   | nova     | enabled | up    | 2022-01-14T00:27:35.000000 |
   +----+----------------+---------------------+----------+---------+-------+----------------------------+

Add the remaining nodes
~~~~~~~~~~~~~~~~~~~~~~~

Now that the single-node cluster is operational, we'll add the
previously-stopped nodes to the cluster. Start them in the reverse order in
which they were stopped.

Here, we will start these nodes (in the order given):

#. ``rabbitmq-server/3``
#. ``rabbitmq-server/1``

Like before, confirm that a node is running after having started it by querying
the good node:

.. code-block:: none

   juju ssh rabbitmq-server/3 sudo systemctl restart rabbitmq-server
   juju ssh rabbitmq-server/2 sudo rabbitmqctl cluster_status
   juju ssh rabbitmq-server/1 sudo systemctl restart rabbitmq-server
   juju ssh rabbitmq-server/2 sudo rabbitmqctl cluster_status

.. note::

   The :command:`systemctl` command is used to start the nodes, as opposed to
   the :command:`rabbitctl start_app`, as the latter command does not apply
   ulimits.

Verify model health
~~~~~~~~~~~~~~~~~~~

It can take at least five minutes for the Juju model to settle to its expected
healthy status. Resolve any hook errors that may appear. Please be patient.

.. note::

   If a node cannot be started (i.e. it's not listed in the ``Running Nodes``
   section of the cluster's status report) it may need to be repaired. See
   cloud operation :doc:`Repair a RabbitMQ node <ops-repair-rabbitmq-node>`.

Verify the model's health with the :command:`juju status rabbitmq-server`
command. Its (partial) output should look like:

.. code-block:: console

   App              Version  Status  Scale  Charm            Store       Channel  Rev  OS      Message
   rabbitmq-server  3.8.2    active      3  rabbitmq-server  charmstore  stable   117  ubuntu  Unit is ready and clustered

   Unit                Workload  Agent  Machine  Public address  Ports     Message
   rabbitmq-server/1*  active    idle   1/lxd/5  10.246.114.55   5672/tcp  Unit is ready and clustered
   rabbitmq-server/2   active    idle   0/lxd/5  10.246.114.56   5672/tcp  Unit is ready and clustered
   rabbitmq-server/3   active    idle   2/lxd/7  10.246.114.59   5672/tcp  Unit is ready and clustered

OpenStack services will auto-reconnect to the cluster once it is in a healthy
state, providing that the IP addresses of the cluster members have not changed.

Verify the resumption of normal cloud operations by running a routine battery
of tests. The creation of a VM is a good choice.

.. LINKS
.. _partitioned: https://www.rabbitmq.com/partitions.html
