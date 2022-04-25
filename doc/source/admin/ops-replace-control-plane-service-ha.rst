:orphan:

======================================
Replace control plane service under HA
======================================

Introduction
------------

The subordinate `hacluster charm`_ is used to provide high availability for
OpenStack control plane applications that lack native (built-in) HA
functionality. This article demonstrates how to replace a node of such a
cluster using Keystone as an example.

.. note::

   Cloud operation :doc:`Implement HA with a VIP <ops-implement-ha-with-vip>`
   shows how to perform an initial deploy of a service under HA.

.. important::

   This procedure will not result in cloud downtime providing that there is at
   least one functional service node present at all times.

Procedure
---------

A node can be first added, followed by the removal of the unwanted node, or the
inverse - removed and then added. This article will use the latter method. One
advantage to doing so is the preservation of the original topology.

As is common for Charmed OpenStack clouds, the Keystone node to be removed is a
containerised unit residing alongside other containerised services on the same
host (hyperconvergence). Removing the unit will therefore remove the container
but not the underlying machine.

List the application units
~~~~~~~~~~~~~~~~~~~~~~~~~~

Display the units, in this case for the keystone application:

.. code-block:: none

   juju status keystone

This article will be based on the following (partial) output:

.. code-block:: console

   App                     Version  Status  Scale  Charm                   Channel      Rev  Exposed  Message
   keystone                21.0.0   active      3  keystone                yoga/edge    566  no       Application Ready
   keystone-hacluster               active      3  hacluster               2.0.3/edge    83  no       Unit is ready and clustered
   keystone-mysql-router   8.0.28   active      3  mysql-router            8.0.19/edge   15  no       Unit is ready

   Unit                         Workload  Agent  Machine  Public address  Ports               Message
   keystone/0*                  active    idle   0/lxd/1  10.246.114.63   5000/tcp            Unit is ready
     keystone-hacluster/0*      active    idle            10.246.114.63                       Unit is ready and clustered
     keystone-mysql-router/0*   active    idle            10.246.114.63                       Unit is ready
   keystone/1                   active    idle   2/lxd/6  10.246.114.80   5000/tcp            Unit is ready
     keystone-hacluster/1       active    idle            10.246.114.80                       Unit is ready and clustered
     keystone-mysql-router/1    active    idle            10.246.114.80                       Unit is ready
   keystone/2                   active    idle   1/lxd/5  10.246.114.79   5000/tcp            Unit is ready
     keystone-hacluster/2       active    idle            10.246.114.79                       Unit is ready and clustered
     keystone-mysql-router/2    active    idle            10.246.114.79                       Unit is ready

In this example, the node to be removed, the unwanted node, will be represented
by unit ``keystone/2``.

Pause the subordinate hacluster unit
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Pause the hacluster unit that corresponds to the principal application unit
being removed. Here, unit ``keystone-hacluster/2`` corresponds to unit
``keystone/2``:

.. code-block:: none

   juju run-action --wait keystone-hacluster/2 pause

Remove the unwanted node
~~~~~~~~~~~~~~~~~~~~~~~~

Remove the unwanted node:

.. code-block:: none

   juju remove-unit keystone/2

This will also remove all subordinate units: ``keystone-hacluster/2`` and
``keystone-mysql-router/2``.

The current state of the model is:

.. code-block:: console

   App                    Version  Status   Scale  Charm         Channel      Rev  Exposed  Message
   keystone               21.0.0   waiting      2  keystone      yoga/edge    566  no       Some units are not ready
   keystone-hacluster              blocked      2  hacluster     2.0.3/edge    83  no       Insufficient peer units for ha cluster (require 3)
   keystone-mysql-router  8.0.28   active       2  mysql-router  8.0.19/edge   15  no       Unit is ready

   Unit                        Workload  Agent  Machine  Public address  Ports     Message
   keystone/0*                 active    idle   0/lxd/1  10.246.114.63   5000/tcp  Unit is ready
     keystone-hacluster/0*     blocked   idle            10.246.114.63             Insufficient peer units for ha cluster (require 3)
     keystone-mysql-router/0*  active    idle            10.246.114.63             Unit is ready
   keystone/1                  active    idle   2/lxd/6  10.246.114.80   5000/tcp  Unit is ready
     keystone-hacluster/1      blocked   idle            10.246.114.80             Insufficient peer units for ha cluster (require 3)
     keystone-mysql-router/1   active    idle            10.246.114.80             Unit is ready

At this time, Keystone will continue to service requests, and the cloud will
remain operational.

Add a principal application unit
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Scale out the existing keystone application and place the new (containerised)
unit on the same host that the removed unit was on (machine 1):

.. code-block:: none

   juju add-unit --to lxd:1 keystone

.. caution::

   If network spaces are in use the above command will not succeed. See Juju
   issue `LP #1969523`_ for a workaround.

It will take a while for the model to settle. Please be patient.

Verify cloud services
~~~~~~~~~~~~~~~~~~~~~

The final :command:`juju status keystone` (partial) output is:

.. code-block:: console

   App                    Version  Status  Scale  Charm         Channel      Rev  Exposed  Message
   keystone               21.0.0   active      3  keystone      yoga/edge    566  no       Application Ready
   keystone-hacluster              active      3  hacluster     2.0.3/edge    83  no       Unit is ready and clustered
   keystone-mysql-router  8.0.28   active      3  mysql-router  8.0.19/edge   15  no       Unit is ready

   Unit                        Workload  Agent  Machine  Public address  Ports     Message
   keystone/0*                 active    idle   0/lxd/1  10.246.114.63   5000/tcp  Unit is ready
     keystone-hacluster/0*     active    idle            10.246.114.63             Unit is ready and clustered
     keystone-mysql-router/0*  active    idle            10.246.114.63             Unit is ready
   keystone/1                  active    idle   2/lxd/6  10.246.114.80   5000/tcp  Unit is ready
     keystone-hacluster/1      active    idle            10.246.114.80             Unit is ready and clustered
     keystone-mysql-router/1   active    idle            10.246.114.80             Unit is ready
   keystone/3                  active    idle   1/lxd/6  10.246.114.79   5000/tcp  Unit is ready
     keystone-hacluster/9      active    idle            10.246.114.79             Unit is ready and clustered
     keystone-mysql-router/15  active    idle            10.246.114.79             Unit is ready

Ensure that all cloud services are working as expected.

.. LINKS
.. _hacluster charm: https://charmhub.io/hacluster
.. _LP #1969523: https://bugs.launchpad.net/juju/+bug/1969523
