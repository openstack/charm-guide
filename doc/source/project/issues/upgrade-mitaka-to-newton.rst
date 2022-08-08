=========================
Upgrade: Mitaka to Newton
=========================

This page contains notes specific to the Mitaka to Newton upgrade path. See the
main :doc:`cdg:upgrade-openstack` page for full coverage.

neutron-gateway charm options
-----------------------------

Between the Mitaka and Newton OpenStack releases, the neutron-gateway charm
added two options, ``bridge-mappings`` and ``data-port``, which replaced the
(now) deprecated ``ext-port`` option. This was to provide for more control over
how ``neutron-gateway`` can configure external networking. Unfortunately, the
charm was only designed to work with either ``ext-port`` (no longer
recommended) *or* ``bridge-mappings`` and ``data-port``.

See bug `LP #1809190`_.

.. BUGS
.. _LP #1809190: https://bugs.launchpad.net/charm-neutron-gateway/+bug/1809190
