===========================
Upgrade: Ussuri to Victoria
===========================

This page contains notes specific to the Ussuri to Victoria upgrade path. See
the main :doc:`../../admin/upgrades/openstack` page for full coverage.

FWaaS project retired
---------------------

The Firewall-as-a-Service (`FWaaS v2`_) OpenStack project is retired starting
with OpenStack Victoria. Consequently, the neutron-api charm will no longer
make this service available starting with that OpenStack release. See the
:doc:`../../release-notes/2110` Release notes on this topic.

Prior to upgrading to Victoria users of FWaaS should remove any existing
firewall groups to avoid the possibility of orphaning active firewalls (see the
`FWaaS v2 CLI documentation`_).

OVN data plane downtime
-----------------------

OVN versions less than 22.03 will lead to downtime of the OVN data plane during
subsequent upgrades. OVN data plane downtime is serious as it will cause
network disruption to all cloud VMs. This is an `upstream OVN issue`_.

Charmed OpenStack supports OVN version 22.03 on Focal cloud nodes. It is
recommended to upgrade directly from any lesser version to 22.03 by following
the procedure detailed on the :doc:`../procedures/ovn-upgrade-2203` page.

.. LINKS
.. _FWaaS v2: https://docs.openstack.org/neutron/ussuri/admin/fwaas.html
.. _FWaaS v2 CLI documentation: https://docs.openstack.org/python-neutronclient/ussuri/cli/osc/v2/firewall-group.html
.. _upstream OVN issue: https://bugs.launchpad.net/charm-ovn-chassis/+bug/1940043
