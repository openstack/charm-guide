:orphan:

=======================
Implement HA with a VIP
=======================

Introduction
------------

The subordinate `hacluster charm`_ provides high availability for OpenStack
applications that lack native (built-in) HA functionality. The clustering
solution is based on Corosync and Pacemaker.

.. important::

   The virtual IP method of implementing HA requires that all units of the
   clustered OpenStack application are on the same subnet.

   The chosen VIP should also be part of a reserved subnet range that MAAS does
   not use for assigning addresses to its nodes.

Procedure
---------

HA can be included during the deployment of the principle application or added
to an existing application.

.. note::

   When hacluster is deployed it is normally given an application name that is
   based on the principle charm name (i.e. <principle-charm-name>-hacluster).

Deploying an application with HA
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

These commands will deploy a three-node Keystone HA cluster with a VIP of
10.246.114.11. Each node will reside in a container on existing machines 0, 1,
and 2:

.. code-block:: none

   juju deploy -n 3 --to lxd:0,lxd:1,lxd:2 --config vip=10.246.114.11 keystone
   juju deploy --config cluster_count=3 hacluster keystone-hacluster
   juju add-relation keystone-hacluster:ha keystone:ha

Adding HA to an existing application
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

These commands will add two units to an assumed single existing unit to create
a three-node Keystone HA cluster with a VIP of 10.246.114.11:

.. code-block:: none

   juju config vip=10.246.114.11 keystone
   juju add-unit -n 2 --to lxd:1,lxd:2 keystone
   juju deploy --config cluster_count=3 hacluster keystone-hacluster
   juju add-relation keystone-hacluster:ha keystone:ha

.. warning::

   Adding HA to an existing application will cause a control plane outage for
   the given application and any applications that depend on it. New units will
   be spawned and the Keystone service catalog will be updated (with the new IP
   address). Plan for a maintenance window.

.. note::

   Adding HA to Keystone in this way is affected by a known issue (tracked in
   bug `LP #1930763`_).

.. LINKS
.. _hacluster charm: https://jaas.ai/hacluster
.. _LP #1930763: https://bugs.launchpad.net/charm-keystone/+bug/1930763
