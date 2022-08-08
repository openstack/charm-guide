===========================
Upgrade: Ussuri to Victoria
===========================

This page contains notes specific to the Ussuri to Victoria upgrade path. See
the main :doc:`cdg:upgrade-openstack` page for full coverage.

FWaaS project retired
---------------------

The Firewall-as-a-Service (`FWaaS v2`_) OpenStack project is retired starting
with OpenStack Victoria. Consequently, the neutron-api charm will no longer
make this service available starting with that OpenStack release. See the
`21.10 release notes`_ on this topic.

Prior to upgrading to Victoria users of FWaaS should remove any existing
firewall groups to avoid the possibility of orphaning active firewalls (see the
`FWaaS v2 CLI documentation`_).

.. LINKS
.. _21.10 release notes: https://docs.openstack.org/charm-guide/latest/2110.html
.. _FWaaS v2: https://docs.openstack.org/neutron/ussuri/admin/fwaas.html
.. _FWaaS v2 CLI documentation: https://docs.openstack.org/python-neutronclient/ussuri/cli/osc/v2/firewall-group.html
