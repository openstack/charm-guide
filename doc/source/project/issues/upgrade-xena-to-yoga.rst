=====================
Upgrade: Xena to Yoga
=====================

This page contains notes specific to the Xena to Yoga upgrade path. See
the main :doc:`../../admin/upgrades/openstack` page for full coverage.

OVN data plane downtime
-----------------------

OVN versions less than 22.03 will lead to downtime of the OVN data plane during
subsequent upgrades. OVN data plane downtime is serious as it will cause
network disruption to all cloud VMs. This is an `upstream OVN issue`_.

Charmed OpenStack supports OVN version 22.03 on Focal cloud nodes. It is
recommended to upgrade directly from any lesser version to 22.03 by following
the procedure detailed on the :doc:`../procedures/ovn-upgrade-2203` page.

.. LINKS
.. _upstream OVN issue: https://bugs.launchpad.net/charm-ovn-chassis/+bug/1940043
