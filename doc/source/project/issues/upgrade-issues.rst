==============
Upgrade issues
==============

This page documents upgrade issues and notes. These may apply to either of the
three upgrade types (charms, OpenStack, series).

.. important::

   It is recommended to read the :doc:`../issues-and-procedures` page before
   continuing.

The issues are organised by upgrade type:

.. contents::
   :local:
   :depth: 2
   :backlinks: top

.. _upgrade_issues_charm_upgrades:

Charm upgrades
--------------

rabbitmq-server charm
~~~~~~~~~~~~~~~~~~~~~

A timing issue has been observed during the upgrade of the rabbitmq-server
charm (see bug `LP #1912638`_ for tracking). If it occurs the resulting hook
error can be resolved with:

.. code-block:: none

   juju resolved rabbitmq-server/N

openstack-dashboard charm: upgrading to revision 294
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When Horizon is configured with TLS (openstack-dashboard charm option
``ssl-cert``) revisions 294 and 295 of the charm have been reported to break
the dashboard (see bug `LP #1853173`_). The solution is to upgrade to a working
revision. A temporary workaround is to disable TLS without upgrading.

.. note::

   Most users will not be impacted by this issue as the recommended approach is
   to always upgrade to the latest revision.

To upgrade to revision 293:

.. code-block:: none

   juju upgrade-charm openstack-dashboard --revision 293

To upgrade to revision 296:

.. code-block:: none

   juju upgrade-charm openstack-dashboard --revision 296

To disable TLS:

.. code-block:: none

   juju config enforce-ssl=false openstack-dashboard

Multiple charms: option ``worker-multiplier``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Starting with OpenStack Charms 21.04 any charm that supports the
``worker-multiplier`` configuration option will, upon upgrade, modify the
active number of service workers according to the following: if the option is
not set explicitly the number of workers will be capped at four regardless of
whether the unit is containerised or not. Previously, the cap applied only to
containerised units.

manila-ganesha charm: package updates
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To fix long-standing issues in the manila-ganesha charm related to `Manila
exporting shares after restart`_, the nfs-ganesha Ubuntu package must be
updated on all affected units prior to the upgrading of the manila-ganesha
charm in OpenStack Charms 21.10.

.. _charm_upgrade_issue-radosgw_gss:

ceph-radosgw charm: upgrading to channel ``quincy/stable``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Due to a `ceph-radosgw charm change`_ in the ``quincy/stable`` channel, URLs
are processed differently by the RADOS Gateway. This will lead to breakage for
an existing ``product-streams`` endpoint, set up by the
glance-simplestreams-sync application, that includes a trailing slash in its
URL.

The glance-simplestreams-sync charm has been fixed in the ``yoga/stable``
channel, but it will not update a pre-existing endpoint. The URL must be
modified (remove the trailing slash) with native OpenStack tooling:

.. code-block:: none

   openstack endpoint list --service product-streams
   openstack endpoint set --url <new-url> <endpoint-id>

.. _upgrade_issues_openstack_upgrades:

OpenStack upgrades
------------------

Nova RPC version mismatches: upgrading Neutron and Nova
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If it is not possible to upgrade Neutron and Nova within the same maintenance
window, be mindful that the RPC communication between nova-cloud-controller,
nova-compute, and nova-api-metadata is very likely to cause several errors
while those services are not running the same version. This is due to the fact
that currently those charms do not support RPC version pinning or
auto-negotiation.

See bug `LP #1825999`_.

Ceph BlueStore mistakenly enabled during OpenStack upgrade
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The Ceph BlueStore storage backend is enabled by default when Ceph Luminous is
detected. Therefore it is possible for a non-BlueStore cloud to acquire
BlueStore by default after an OpenStack upgrade (Luminous first appeared in
Queens). Problems will occur if storage is scaled out without first disabling
BlueStore (set ceph-osd charm option ``bluestore`` to 'False'). See bug `LP
#1885516`_ for details.

.. _ceph-require-osd-release:

Ceph: option ``require-osd-release``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Before upgrading Ceph, option ``require-osd-release`` should reflect the
current Ceph release (e.g. 'nautilus' if upgrading to Octopus). Otherwise, the
subsequent upgrade may fail, rendering the cluster inoperable.

On any ceph-mon unit, the current value of the option can be queried with:

.. code-block:: none

   sudo ceph osd dump | grep require_osd_release

If it needs changing, it can be done manually on any ceph-mon unit. Here the
current release is Nautilus:

.. code-block:: none

   sudo ceph osd require-osd-release nautilus

Once Ceph (all charms) is upgraded, the option should be set to the new
release. Here the new release is Octopus:

.. code-block:: none

   sudo ceph osd require-osd-release octopus

Bug `LP #1929254`_ is for tracking this effort.

Octavia
~~~~~~~

An Octavia upgrade may entail an update of its load balancers (amphorae) as a
post-upgrade task. Reasons for doing this include:

* API incompatibility between the amphora agent and the new Octavia service
* the desire to use features available in the new amphora agent or haproxy

See the upstream documentation on `Rotating amphora images`_.

.. _upgrade_issues_series_upgrades:

Series upgrades
---------------

DNS HA: upgrade to focal
~~~~~~~~~~~~~~~~~~~~~~~~

DNS HA has been reported to not work on the focal series. See `LP #1882508`_
for more information.

LXD container upgrade to jammy
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

While performing LXD container series upgrades from focal to jammy, these
containers may lose their IP addresses and network connectivity on reboot
due to `LP #2041480`_.

This issue currently only affects juju deployed LXD containers where deployed
using juju 2.9.43 and older. LXD containers deployed using juju 2.9.44+
are not effected.

Please see the bug report for more details and available workarounds.

It is highly recommended to apply the workarounds prior to performing the
series upgrades.


Upgrading while Vault is sealed
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If a series upgrade is attempted while Vault is sealed then manual intervention
will be required (see bugs `LP #1886083`_ and `LP #1890106`_). The vault leader
unit (which will be in error) will need to be unsealed and the hook error
resolved. The `vault charm`_ README has unsealing instructions, and the hook
error can be resolved with:

.. code-block:: none

   juju resolved vault/N

.. LINKS
.. _Release Notes: https://docs.openstack.org/charm-guide/latest/release-notes.html
.. _Ubuntu Cloud Archive: https://wiki.ubuntu.com/OpenStack/CloudArchive
.. _Upgrades: https://docs.openstack.org/operations-guide/ops-upgrades.html
.. _Update services: https://docs.openstack.org/operations-guide/ops-upgrades.html#update-services
.. _Various issues: various-issues.html
.. _Special charm procedures: upgrade-special.html
.. _vault charm: https://opendev.org/openstack/charm-vault/src/branch/master/src/README.md#unseal-vault
.. _manila exporting shares after restart: https://bugs.launchpad.net/charm-manila-ganesha/+bug/1889287
.. _Rotating amphora images: https://docs.openstack.org/octavia/latest/admin/guides/operator-maintenance.html#rotating-the-amphora-images
.. _ceph-radosgw charm change: https://review.opendev.org/c/openstack/charm-ceph-radosgw/+/835827

.. BUGS
.. _LP #1825999: https://bugs.launchpad.net/charm-nova-compute/+bug/1825999
.. _LP #1853173: https://bugs.launchpad.net/charm-openstack-dashboard/+bug/1853173
.. _LP #1882508: https://bugs.launchpad.net/charm-deployment-guide/+bug/1882508
.. _LP #1885516: https://bugs.launchpad.net/charm-deployment-guide/+bug/1885516
.. _LP #1886083: https://bugs.launchpad.net/vault-charm/+bug/1886083
.. _LP #1890106: https://bugs.launchpad.net/vault-charm/+bug/1890106
.. _LP #1912638: https://bugs.launchpad.net/charm-rabbitmq-server/+bug/1912638
.. _LP #1929254: https://bugs.launchpad.net/charm-ceph-osd/+bug/1929254
.. _LP #2041480: https://bugs.launchpad.net/ubuntu/+source/netplan.io/+bug/2041480
