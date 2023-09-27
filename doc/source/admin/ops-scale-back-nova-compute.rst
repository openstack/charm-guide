:orphan:

=======================================
Scale back the nova-compute application
=======================================

.. important::

   This page has been identified as being affected by the breaking changes
   introduced between versions 2.9.x and 3.x of the Juju client. Read
   support note :ref:`juju_29_3x_changes` before continuing.

Preamble
--------

Scaling in the nova-compute application implies the removal of one or more
nova-compute units (i.e. compute nodes). This is easily done with generic Juju
commands and actions available to the nova-compute charm.

Procedure
---------

List the nova-compute units
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Display the nova-compute units:

.. code-block:: none

   juju status nova-compute

This article will be based on the following output:

.. code-block:: console

   Unit              Workload  Agent  Machine  Public address  Ports    Message
   nova-compute/0*   active    idle   15       10.5.0.5                 Unit is ready
     ntp/0*          active    idle            10.5.0.5        123/udp  chrony: Ready
     ovn-chassis/0*  active    idle            10.5.0.5                 Unit is ready
   nova-compute/1    active    idle   16       10.5.0.24                Unit is ready
     ntp/2           active    idle            10.5.0.24       123/udp  chrony: Ready
     ovn-chassis/2   active    idle            10.5.0.24                Unit is ready
   nova-compute/2    active    idle   17       10.5.0.10                Unit is ready
     ntp/1           active    idle            10.5.0.10       123/udp  chrony: Ready
     ovn-chassis/1   active    idle            10.5.0.10                Unit is ready

.. tip::

   You can use the :command:`openstack` client to map compute nodes to
   nova-compute units by IP address: ``openstack hypervisor list``.

Disable the node
~~~~~~~~~~~~~~~~

Disable the compute node by referring to its corresponding unit, here
``nova-compute/0``:

.. code-block:: none

   juju run nova-compute/0 disable

This will stop nova-compute services and inform nova-scheduler to no longer
assign new VMs to the unit.

.. warning::

   Before continuing, make sure that all VMs hosted on the target compute node
   have been either deleted or migrated to another node.

.. _unregister_compute_node:

Unregister the node
~~~~~~~~~~~~~~~~~~~

Now unregister the compute node from the cloud:

.. code-block:: none

   juju run nova-compute/0 remove-from-cloud

The workload status of the unit can be checked with:

.. code-block:: none

   juju status nova-compute/0

Sample output:

.. code-block:: console

   Unit              Workload  Agent  Machine  Public address  Ports    Message
   nova-compute/0*   blocked   idle   15       10.5.0.5                 Unit was removed from the cloud
     ntp/0*          active    idle            10.5.0.5        123/udp  chrony: Ready
     ovn-chassis/0*  active    idle            10.5.0.5                 Unit is ready

At this point (before the unit is actually removed from the model with the
:command:`remove-unit` command) the process can be reverted with the
``register-to-cloud`` action, followed by the ``enable`` action. This
combination will restart nova-compute services and enable nova-scheduler to run
new VMs on the unit.

Remove the unit
~~~~~~~~~~~~~~~

Now that the compute node has been logically removed at the OpenStack level,
remove its unit from the model:

.. code-block:: none

   juju remove-unit nova-compute/0

Request the status of the application once more:

.. code-block:: none

   juju status nova-compute

The unit's removal should be confirmed by its absence in the output:

.. code-block:: console

   Unit              Workload  Agent  Machine  Public address  Ports    Message
   nova-compute/1*   active    idle   16       10.5.0.24                Unit is ready
     ntp/2*          active    idle            10.5.0.24       123/udp  chrony: Ready
     ovn-chassis/2   active    idle            10.5.0.24                Unit is ready
   nova-compute/2    active    idle   17       10.5.0.10                Unit is ready
     ntp/1           active    idle            10.5.0.10       123/udp  chrony: Ready
     ovn-chassis/1*  active    idle            10.5.0.10                Unit is ready
