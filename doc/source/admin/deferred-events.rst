=======================
Deferred service events
=======================

.. important::

   This page has been identified as being affected by the breaking changes
   introduced between versions 2.9.x and 3.x of the Juju client. Read
   support note :ref:`juju_29_3x_changes` before continuing.

Overview
--------

Operational or maintenance procedures applied to a cloud often lead to the
restarting of various OpenStack services and/or the calling of certain charm
hooks. Although normal, such events can be undesirable due to the service
interruptions they can cause.

The deferred service events feature allows the operator the choose to prevent
these service restarts and hook calls from occurring. These deferred events can
then be resolved by the operator at a more opportune time.

Situations in which these service events are prone to take place include:

* charm upgrades
* OpenStack upgrades (charm-managed)
* package upgrades (non-charm-managed)
* charm configuration option changes

Charms
------

Deferred restarts are supported on a per-charm basis. This support will be
mentioned in a charm's README along with any charm-specific deferred restart
information.

Here is the current list of deferred restart-aware charms:

* `neutron-gateway`_
* `neutron-openvswitch`_
* `ovn-central`_
* `ovn-chassis`_
* `ovn-dedicated-chassis`_
* `rabbitmq-server`_

Enabling and disabling deferred service events
----------------------------------------------

Deferred service events are disabled by default for all charms. To enable them
for a charm:

.. code-block:: none

   juju config <charm-name> enable-auto-restarts=False

.. important::

   The ``enable-auto-restarts`` option can only be set post-deployment.

To disable deferred service events for a charm:

.. code-block:: none

   juju config <charm-name> enable-auto-restarts=True

Identifying and resolving deferred service events
-------------------------------------------------

The existence of a deferred service event is exposed in the output of the
:command:`juju status` command. The following two sections provide an example
of how to identify and resolve each type.

Service restarts
~~~~~~~~~~~~~~~~

Here the ``neutron-openvswitch/1`` unit is affected by a deferred service
restart:

.. code-block:: console

   App                  Version  Status  Scale  Charm                Store       Channel  Rev  OS      Message
   neutron-openvswitch  16.3.0   active      2  neutron-openvswitch  charmstore           433  ubuntu  Unit is ready
   nova-compute         21.1.2   active      2  nova-compute         charmstore           537  ubuntu  Unit is ready.

   Unit                      Workload  Agent  Machine  Public address  Ports  Message
   nova-compute/0*           active    idle   6        172.20.0.13            Unit is ready.
     neutron-openvswitch/1   active    idle            172.20.0.13            Unit is ready. Services queued for restart: openvswitch-switch
   nova-compute/1            active    idle   7        172.20.0.4             Unit is ready
     neutron-openvswitch/0*  active    idle            172.20.0.4             Unit is ready

To see more detail the ``show-deferred-events`` action is used:

.. code-block:: none

   juju run --wait neutron-openvswitch/1 show-deferred-events

   unit-neutron-openvswitch-1:
     UnitId: neutron-openvswitch/1
     id: "67"
     results:
       Stdout: |
         none
       output: |
         hooks: []
         restarts:
         - 1618896650 openvswitch-switch                       Package update
     status: completed
     timing:
       completed: 2021-04-20 05:52:39 +0000 UTC
       enqueued: 2021-04-20 05:52:32 +0000 UTC
       started: 2021-04-20 05:52:33 +0000 UTC

In this example, the message "Package update" is displayed. This signifies that
the package management software of the host is responsible for the service
restart request.

Resolving deferred service restarts
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To resolve a deferred service restart on a unit run the ``restart-services``
action:

.. code-block:: none

   juju run --wait neutron-openvswitch/1 restart-services deferred-only=True

The argument ``deferred-only`` ensures that only the necessary services are
restarted (for a charm that manages multiple services).

.. note::

   Alternatively, the service can be restarted manually on the unit. The status
   message will be removed in due course by the charm (i.e. during the next
   ``update-status`` hook execution - a maximum delay of five minutes).

Hook calls
~~~~~~~~~~

Here the ``neutron-openvswitch/1`` unit is affected by a deferred hook call:

.. code-block:: console

   App                  Version  Status  Scale  Charm                Store       Channel  Rev  OS      Message
   neutron-openvswitch  16.3.0   active      2  neutron-openvswitch  charmstore           433  ubuntu  Unit is ready. Hooks skipped due to disabled auto restarts: config-changed
   nova-compute         21.1.2   active      2  nova-compute         charmstore           537  ubuntu  Unit is ready

   Unit                      Workload  Agent  Machine  Public address  Ports  Message
   nova-compute/0*           active    idle   6        172.20.0.13            Unit is ready
     neutron-openvswitch/1   active    idle            172.20.0.13            Unit is ready. Hooks skipped due to disabled auto restarts: config-changed

Resolving deferred hook calls
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To resolve a deferred hook call on a unit run the ``run-deferred-hooks``
action:

.. code-block:: none

   juju run --wait neutron-openvswitch/1 run-deferred-hooks

.. LINKS

.. CHARMS
.. _neutron-gateway: https://opendev.org/openstack/charm-neutron-gateway/src/branch/master/README.md#deferred-service-events
.. _neutron-openvswitch: https://opendev.org/openstack/charm-neutron-openvswitch/src/branch/master/README.md#deferred-service-events
.. _ovn-central: https://opendev.org/x/charm-ovn-central/src/branch/master/README.md#deferred-service-events
.. _ovn-chassis: https://opendev.org/x/charm-ovn-chassis/src/branch/master/README.md#deferred-service-events
.. _ovn-dedicated-chassis: https://opendev.org/x/charm-ovn-dedicated-chassis/src/branch/master/README.md#deferred-service-events
.. _rabbitmq-server: https://opendev.org/openstack/charm-rabbitmq-server/src/branch/master/README.md#deferred-service-events
