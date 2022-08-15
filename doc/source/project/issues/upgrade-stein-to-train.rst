=======================
Upgrade: Stein to Train
=======================

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

   juju run --application=memcached 'sudo systemctl restart memcached'

.. BUGS
.. _LP #1828534: https://bugs.launchpad.net/charm-designate/+bug/1828534
.. _LP #1928992: https://bugs.launchpad.net/charm-deployment-guide/+bug/1928992
