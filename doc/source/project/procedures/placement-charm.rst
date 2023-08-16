===========================================
placement charm: OpenStack upgrade to Train
===========================================

.. important::

   This page has been identified as being affected by the breaking changes
   introduced between versions 2.9.x and 3.x of the Juju client. Read
   support note :ref:`juju_29_3x_changes` before continuing.

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
   juju run --wait nova-cloud-controller/leader pause
   juju integrate placement percona-cluster
   juju integrate placement keystone
   juju integrate placement nova-cloud-controller

List endpoints and ensure placement endpoints are now listening on the new
placement IP address. Follow this up by resuming nova-cloud-controller:

.. code-block:: none

   openstack endpoint list
   juju run --wait nova-cloud-controller/leader resume

Finally, upgrade the nova-cloud-controller services. Below all units are
upgraded simultaneously but see the :ref:`paused_single_unit` service upgrade
method for a more controlled approach:

.. code-block:: none

   juju config nova-cloud-controller openstack-origin=cloud:bionic-train

The Compute service (nova-compute) should then be upgraded.

.. LINKS
.. _placement: https://charmhub.io/placement
