=======================
Upgrade: Stein to Train
=======================

.. important::

   This page has been identified as being affected by the breaking changes
   introduced between versions 2.9.x and 3.x of the Juju client. Read
   support note :ref:`juju_29_3x_changes` before continuing.

This page contains notes specific to the Stein to Train upgrade path. See the
main :doc:`../../admin/upgrades/openstack` page for full coverage.

New placement charm
-------------------

This upgrade path requires the inclusion of a new charm. Please see special
charm procedure :doc:`../procedures/placement-charm`.

Placement endpoints not updated in Keystone service catalogue
-------------------------------------------------------------

When the placement charm is deployed during the upgrade to Train (as per the
above section) the Keystone service catalogue is not updated accordingly. This
issue is tracked in bug `LP #1928992`_, which also includes an explicit
workaround (comment #4).

Neutron LBaaS retired
---------------------

As of Train, support for Neutron LBaaS has been retired. The load-balancing
services are now provided by Octavia LBaaS. There is no automatic migration
path, please review the :doc:`../../admin/networking/load-balancing` page in
the Charm Guide for more information.

Designate encoding issue
------------------------

When upgrading Designate to Train, there is an encoding issue between the
designate-producer and memcached that causes the designate-producer to crash.
See bug `LP #1828534`_. This can be resolved by restarting the memcached service.

.. code-block:: none

   juju exec --application=memcached 'sudo systemctl restart memcached'

Ceph PG auto-scaler not enabled by default
------------------------------------------

The upgrade to Train implies an upgrade of Ceph from Mimic to Nautilus, which
in turn includes the new `PG auto-scaler`_ Ceph module. However, this new
feature is enabled by default only for new deployments. To enable it for an
upgraded (to Nautilus) cluster:

.. code-block:: none

   juju config ceph-mon pg-autotune=true

.. LINKS
.. _PG auto-scaler: https://ceph.io/en/news/blog/2019/new-in-nautilus-pg-merging-and-autotuning

.. BUGS
.. _LP #1828534: https://bugs.launchpad.net/charm-designate/+bug/1828534
.. _LP #1928992: https://bugs.launchpad.net/charm-deployment-guide/+bug/1928992
