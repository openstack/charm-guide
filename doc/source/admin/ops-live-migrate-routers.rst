======================================
Live migrate Neutron routers (OVS/ML2)
======================================

Introduction
------------

Neutron routers are hosted by ``L3 Agents`` running on `neutron-gateway`_
units. When the unit(s) need to be brought down for maintenance, routers need
to be migrated to another unit that will host them during the downtime.

.. note::

   This tutorial deals with migration of a router strictly in the OVS/ML2
   environment, deployments using OVN have different topology in which routers
   are not hosted by individual ``L3 Agents``.

.. warning::

   Migration of a router will cause brief disruption of a traffic between
   networks connected to the router.

Example topology
----------------

To demonstrate steps needed for the migration, we will use openstack deployment
with three `neutron-gateway`_ units:

.. code-block:: console

   Unit                Workload  Agent  Machine  Public address  Ports    Message
   neutron-gateway/0*  active    idle   0        10.5.1.149               Unit is ready
     ntp/0*            active    idle            10.5.1.149      123/udp  chrony: Ready
   neutron-gateway/1   active    idle   13       10.5.2.17                Unit is ready
     ntp/4             active    idle            10.5.2.17       123/udp  chrony: Ready
   neutron-gateway/2   active    idle   14       10.5.0.159               Unit is ready
     ntp/5             active    idle            10.5.0.159      123/udp  chrony: Ready

   Machine  State    DNS         Inst id                               Series  AZ    Message
   0        started  10.5.1.149  1bf707d8-3e27-473f-b597-5cfc43c50b85  bionic  nova  ACTIVE
   13       started  10.5.2.17   960d3868-a5a3-443c-99a8-acbcec7a4272  bionic  nova  ACTIVE
   14       started  10.5.0.159  c8c75144-7a5b-49c0-bfb5-4910c6501f21  bionic  nova  ACTIVE

Each of these units hosts single ``L3 Agent``:

.. code-block:: console

   openstack network agent list --agent-type l3

   +--------------------------------------+------------+-------------------+-------------------+-------+-------+------------------+
   | ID                                   | Agent Type | Host              | Availability Zone | Alive | State | Binary           |
   +--------------------------------------+------------+-------------------+-------------------+-------+-------+------------------+
   | 6226d159-ed57-4dfb-a311-ae36d2b4462b | L3 agent   | juju-56ea63-os-0  | nova              | :-)   | UP    | neutron-l3-agent |
   | 6ae7ed32-7aa4-460c-8eb7-d13f4e651fcc | L3 agent   | juju-56ea63-os-13 | nova              | :-)   | UP    | neutron-l3-agent |
   | b066c34d-aa91-46fe-9b20-f19f4a297932 | L3 agent   | juju-56ea63-os-14 | nova              | :-)   | UP    | neutron-l3-agent |
   +--------------------------------------+------------+-------------------+-------------------+-------+-------+------------------+

And we will  be bringing down unit ``neutron-gateway/2`` for maintenance.

Procedure
---------

Find out which routers run on our targeted unit
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Referencing our example topology above, we are about to bring down unit
``neutron-gateway/2``. We need to find out which routers are hosted on it.
We can see that this unit is running on machine ``14``, listing all the
L3 Agents we get:

.. code-block:: console

   openstack network agent list --agent-type l3

   +--------------------------------------+------------+-------------------+-------------------+-------+-------+------------------+
   | ID                                   | Agent Type | Host              | Availability Zone | Alive | State | Binary           |
   +--------------------------------------+------------+-------------------+-------------------+-------+-------+------------------+
   | 6226d159-ed57-4dfb-a311-ae36d2b4462b | L3 agent   | juju-56ea63-os-0  | nova              | :-)   | UP    | neutron-l3-agent |
   | 6ae7ed32-7aa4-460c-8eb7-d13f4e651fcc | L3 agent   | juju-56ea63-os-13 | nova              | :-)   | UP    | neutron-l3-agent |
   | b066c34d-aa91-46fe-9b20-f19f4a297932 | L3 agent   | juju-56ea63-os-14 | nova              | :-)   | UP    | neutron-l3-agent |
   +--------------------------------------+------------+-------------------+-------------------+-------+-------+------------------+

Machine ``14`` corresponds to Host ``juju-56ea63-os-14``, so we need to find
out which routers are hosted on the ``L3 agent`` with ID
``b066c34d-aa91-46fe-9b20-f19f4a297932``.

.. code-block:: console

   openstack router list --agent b066c34d-aa91-46fe-9b20-f19f4a297932

   +--------------------------------------+-----------------+--------+-------+-------------+-------+----------------------------------+
   | ID                                   | Name            | Status | State | Distributed | HA    | Project                          |
   +--------------------------------------+-----------------+--------+-------+-------------+-------+----------------------------------+
   | 35dcce1b-f69f-4af3-b46f-54249812ec9f | provider-router | ACTIVE | UP    | False       | False | b800e60c5e3841fbbb8b1dbc02ce13e3 |
   +--------------------------------------+-----------------+--------+-------+-------------+-------+----------------------------------+

This means that we need to move router with name ``provider-router`` to a
different agent.

Move router to a different agent
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This is the part that will cause brief disruption of a traffic as routers can
not be moved seamlessly. Moving consists of manually removing router from one
agent and adding it to another. First we double-check that the router is hosted
on the agent that is about to go down:

.. code-block:: console

   openstack network agent list --router provider-router

   +--------------------------------------+------------+-------------------+-------------------+-------+-------+------------------+
   | ID                                   | Agent Type | Host              | Availability Zone | Alive | State | Binary           |
   +--------------------------------------+------------+-------------------+-------------------+-------+-------+------------------+
   | b066c34d-aa91-46fe-9b20-f19f4a297932 | L3 agent   | juju-56ea63-os-14 | nova              | :-)   | UP    | neutron-l3-agent |
   +--------------------------------------+------------+-------------------+-------------------+-------+-------+------------------+

Now we remove the router from this agent and assign it to an agent running on
the unit ``neutron-gateway/0`` with agent ID
``6226d159-ed57-4dfb-a311-ae36d2b4462b``. Note that this will increase the load
on the ``neutron-gateway/0`` unit. Make sure that the machine hosting this unit
has enough resources to accommodate the additional router.

.. code-block:: console

   openstack network agent remove router b066c34d-aa91-46fe-9b20-f19f4a297932 provider-router --l3
   openstack network agent add router 6226d159-ed57-4dfb-a311-ae36d2b4462b provider-router --l3

Result of this move can be checked by:

.. code-block:: console

   openstack network agent list --router provider-router

   +--------------------------------------+------------+------------------+-------------------+-------+-------+------------------+
   | ID                                   | Agent Type | Host             | Availability Zone | Alive | State | Binary           |
   +--------------------------------------+------------+------------------+-------------------+-------+-------+------------------+
   | 6226d159-ed57-4dfb-a311-ae36d2b4462b | L3 agent   | juju-56ea63-os-0 | nova              | :-)   | UP    | neutron-l3-agent |
   +--------------------------------------+------------+------------------+-------------------+-------+-------+------------------+

.. LINKS
.. _neutron-gateway : https://charmhub.io/neutron-gateway
