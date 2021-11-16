:orphan:

=======================
Replace a RabbitMQ node
=======================

Introduction
~~~~~~~~~~~~

There may be times when a cloud's RabbitMQ cluster experiences difficulties.
When this happens one option to bringing the cluster back to a healthy state is
to replace one of its nodes.

.. important::

   This procedure will not result in cloud downtime providing that there is at
   least one functional RabbitMQ node present.

   However, it will trigger the restart of every client application (cloud
   service) that has a relation to the rabbitmq-server application. This may
   have adverse effects on the cloud's ability to service its current workload.

   The above issue can be mitigated through the use of :doc:`Deferred service
   events <../howto/deferred-events>`. If restarts are deferred, a client will
   nonetheless experience a slight delay (due to timeouts) if it tries to
   contact a non-existent node.

.. note::

   An alternative approach to replacing a node is to repair it. See operation
   :doc:`Repair a RabbitMQ node <ops-repair-rabbitmq-node>`, which includes a
   comparison of the repair and replacement methods.

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

In this example the unhealthy node is assumed to be ``rabbitmq-server/0`` with
an arbitrary message indicative of a problem. This is the node that is to be
replaced.

Replace the unhealthy node
~~~~~~~~~~~~~~~~~~~~~~~~~~

First remove the unhealthy node:

.. code-block:: none

   juju remove-unit rabbitmq-server/0

Then add a node. Here we will be deploying to a new container on existing
machine '5':

.. code-block:: none

   juju add-unit --to lxd:5 rabbitmq-server

Please be patient as the node joins the cluster. In this example a machine (a
LXD container) must also be provisioned.

Verify model health
~~~~~~~~~~~~~~~~~~~

Verify the model's health with the :command:`juju status rabbitmq-server`
command. Its partial output should look like:

.. code-block:: console

   App              Version  Status  Scale  Charm            Store       Channel  Rev  OS      Message
   rabbitmq-server  3.8.2    active      3  rabbitmq-server  charmstore  stable   118  ubuntu  Unit is ready and clustered

   Unit                Workload  Agent  Machine  Public address  Ports     Message
   rabbitmq-server/0   active    idle   5/lxd/1  10.0.0.178      5672/tcp  Unit is ready and clustered
   rabbitmq-server/1*  active    idle   1/lxd/0  10.0.0.175      5672/tcp  Unit is ready and clustered
   rabbitmq-server/2   active    idle   2/lxd/0  10.0.0.176      5672/tcp  Unit is ready and clustered
