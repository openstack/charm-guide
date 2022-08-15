===========================================
placement charm: OpenStack upgrade to Train
===========================================

.. note::

   This page describes a procedure that is required when performing an upgrade
   of an OpenStack cloud. Please read the more general
   :doc:`../../admin/upgrades/overview` before attempting any of the
   instructions given here.

As of OpenStack Train, the Placement API is managed by the new `placement`_
charm and is no longer managed by the nova-cloud-controller charm. The upgrade
to Train therefore involves some coordination to transition to the new API
endpoints.

Prior to upgrading nova-cloud-controller services to Train, the placement charm
must be deployed for Train and related to the Stein-based nova-cloud-controller
application. It is important that the nova-cloud-controller unit leader is
paused while the API transition occurs (paused prior to adding relations for
the placement charm) as the placement charm will migrate existing placement
tables from the nova_api database to a new placement database. Once the new
placement endpoints are registered, nova-cloud-controller can be resumed.

Here are example commands for the process just described:

.. code-block:: none

   juju deploy --series bionic --config openstack-origin=cloud:bionic-train cs:placement
   juju run-action --wait nova-cloud-controller/leader pause
   juju add-relation placement percona-cluster
   juju add-relation placement keystone
   juju add-relation placement nova-cloud-controller

List endpoints and ensure placement endpoints are now listening on the new
placement IP address. Follow this up by resuming nova-cloud-controller:

.. code-block:: none

   openstack endpoint list
   juju run-action --wait nova-cloud-controller/leader resume

Finally, upgrade the nova-cloud-controller services. Below all units are
upgraded simultaneously but see the :ref:`paused_single_unit` service upgrade
method for a more controlled approach:

.. code-block:: none

   juju config nova-cloud-controller openstack-origin=cloud:bionic-train

The Compute service (nova-compute) should then be upgraded.

.. LINKS
.. _placement: https://charmhub.io/placement
