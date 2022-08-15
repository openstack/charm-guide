==============
Charms upgrade
==============

The Juju command to use is :command:`upgrade-charm`. For extra guidance see
`How to upgrade applications`_ in the Juju documentation.

Please read the following before continuing:

* :doc:`overview`
* :doc:`../../release-notes/index`
* :doc:`../../project/issues-and-procedures`

.. note::

   A charm upgrade affects all corresponding units; upgrading on a per-unit
   basis is not currently supported.

Upgrade order
-------------

There is no special order in which to upgrade the charms. The order described
here is based on the upgrade order for :ref:`OpenStack upgrades
<openstack_upgrade_order>`, which, in turn, is the order used by internal
testing.

.. note::

   Although it may be possible to upgrade some charms concurrently it is
   recommended that charm upgrades be performed sequentially (i.e. one at a
   time). Verify a charm upgrade before moving on to the next.

The general order is:

#. all principal charms
#. all subordinate charms

The precise order within the group of principal charms is shown in the below
table.

.. note::

   At this time, only stable charms are listed in the upgrade order table.

.. list-table:: Principal charms
   :header-rows: 1
   :widths: auto

   * - Order
     - Charm

   * - 1
     - `percona-cluster`_ or `mysql-innodb-cluster`_

   * - 2
     - `rabbitmq-server`_

   * - 3
     - `ceph-mon`_

   * - 4
     - `keystone`_

   * - 5
     - `aodh`_

   * - 6
     - `barbican`_

   * - 7
     - `ceilometer`_

   * - 8
     - `ceph-fs`_

   * - 9
     - `ceph-radosgw`_

   * - 10
     - `cinder`_

   * - 11
     - `designate`_

   * - 12
     - `designate-bind`_

   * - 13
     - `glance`_

   * - 14
     - `gnocchi`_

   * - 15
     - `heat`_

   * - 16
     - `manila`_

   * - 17
     - `manila-ganesha`_

   * - 18
     - `neutron-api`_

   * - 19
     - `neutron-gateway`_ or `ovn-dedicated-chassis`_

   * - 20
     - `ovn-central`_

   * - 21
     - `placement`_

   * - 22
     - `nova-cloud-controller`_

   * - 23
     - `nova-compute`_

   * - 24
     - `openstack-dashboard`_

   * - 25
     - `ceph-osd`_

   * - 26
     - `swift-proxy`_

   * - 27
     - `swift-storage`_

   * - 28
     - `octavia`_

Upgrade testing for subordinate charms does not follow a prescribed order. Once
all the principal charms have been processed all the subordinate charms can
then be upgraded in any order.

Perform the upgrade
-------------------

Prior to upgrading a charm, say keystone, a (partial) output to :command:`juju
status` may look like:

.. code-block:: console

   App       Version  Status   Scale  Charm     Store       Rev  OS      Notes
   keystone  15.0.0   active       1  keystone  jujucharms  306  ubuntu

   Unit             Workload  Agent  Machine  Public address  Ports      Message
   keystone/0*      active    idle   3/lxd/1  10.248.64.69    5000/tcp   Unit is ready

Here, as deduced from the Keystone **service** version of '15.0.0', the cloud
is running Stein. The 'keystone' **charm** however shows a revision number of
'306'. Upon charm upgrade, the service version will remain unchanged but the
charm revision is expected to increase in number.

So to upgrade the keystone charm:

.. code-block:: none

   juju upgrade-charm keystone

The upgrade progress can be monitored via :command:`juju status`. Any
encountered problem will surface as a message in its output. This sample
(partial) output reflects a successful upgrade:

.. code-block:: console

   App       Version  Status   Scale  Charm     Store       Rev  OS      Notes
   keystone  15.0.0   active       1  keystone  jujucharms  309  ubuntu

   Unit             Workload  Agent  Machine  Public address  Ports      Message
   keystone/0*      active    idle   3/lxd/1  10.248.64.69    5000/tcp   Unit is ready

This shows that the charm now has a revision number of '309' but Keystone
itself remains at '15.0.0'.

.. caution::

   Any software changes that may have (exceptionally) been made to a charm
   currently running on a unit will be overwritten by the target charm during
   the upgrade.

Upgrade target revisions
~~~~~~~~~~~~~~~~~~~~~~~~

By default the :command:`upgrade-charm` command will upgrade a charm to its
latest stable revision (a possible multi-step upgrade). This means that
intervening revisions can be conveniently skipped. Use the ``--revision``
option to specify a target revision.

The current revision can be discovered via :command:`juju status` output (see
column 'Rev'). For the ceph-mon charm:

.. code-block:: console

   App       Version  Status  Scale  Charm     Store       Rev  OS      Notes
   ceph-mon  13.2.8   active      3  ceph-mon  jujucharms   48  ubuntu

.. important::

   As stated earlier, any kind of upgrade should first be tested in a
   pre-production environment. OpenStack charm upgrades have been tested for
   single-step upgrades only (N+1).

.. LINKS
.. _How to upgrade applications: https://juju.is/docs/olm/upgrade-applications
.. _Release Notes: https://docs.openstack.org/charm-guide/latest/release-notes.html

.. _aodh: https://opendev.org/openstack/charm-aodh/
.. _barbican: https://opendev.org/openstack/charm-barbican/
.. _barbican-vault: https://opendev.org/openstack/charm-barbican-vault/
.. _ceilometer: https://opendev.org/openstack/charm-ceilometer/
.. _ceilometer-agent: https://opendev.org/openstack/charm-ceilometer-agent/
.. _cinder: https://opendev.org/openstack/charm-cinder/
.. _cinder-backup: https://opendev.org/openstack/charm-cinder-backup/
.. _cinder-backup-swift-proxy: https://opendev.org/openstack/charm-cinder-backup-swift-proxy/
.. _cinder-ceph: https://opendev.org/openstack/charm-cinder-ceph/
.. _designate: https://opendev.org/openstack/charm-designate/
.. _glance: https://opendev.org/openstack/charm-glance/
.. _heat: https://opendev.org/openstack/charm-heat/
.. _keystone: https://opendev.org/openstack/charm-keystone/
.. _keystone-ldap: https://opendev.org/openstack/charm-keystone-ldap/
.. _keystone-saml-mellon: https://opendev.org/openstack/charm-keystone-saml-mellon/
.. _manila: https://opendev.org/openstack/charm-manila/
.. _manila-ganesha: https://opendev.org/openstack/charm-manila-ganesha/
.. _masakari: https://opendev.org/openstack/charm-masakari/
.. _masakari-monitors: https://opendev.org/openstack/charm-masakari-monitors/
.. _mysql-innodb-cluster: https://opendev.org/openstack/charm-mysql-innodb-cluster
.. _mysql-router: https://opendev.org/openstack/charm-mysql-router
.. _neutron-api: https://opendev.org/openstack/charm-neutron-api/
.. _neutron-api-plugin-arista: https://opendev.org/openstack/charm-neutron-api-plugin-arista
.. _neutron-api-plugin-ovn: https://opendev.org/openstack/charm-neutron-api-plugin-ovn
.. _neutron-dynamic-routing: https://opendev.org/openstack/charm-neutron-dynamic-routing/
.. _neutron-gateway: https://opendev.org/openstack/charm-neutron-gateway/
.. _neutron-openvswitch: https://opendev.org/openstack/charm-neutron-openvswitch/
.. _nova-cell-controller: https://opendev.org/openstack/charm-nova-cell-controller/
.. _nova-cloud-controller: https://opendev.org/openstack/charm-nova-cloud-controller/
.. _nova-compute: https://opendev.org/openstack/charm-nova-compute/
.. _octavia: https://opendev.org/openstack/charm-octavia/
.. _octavia-dashboard: https://opendev.org/openstack/charm-octavia-dashboard/
.. _octavia-diskimage-retrofit: https://opendev.org/openstack/charm-octavia-diskimage-retrofit/
.. _openstack-dashboard: https://opendev.org/openstack/charm-openstack-dashboard/
.. _placement: https://opendev.org/openstack/charm-placement
.. _swift-proxy: https://opendev.org/openstack/charm-swift-proxy/
.. _swift-storage: https://opendev.org/openstack/charm-swift-storage/

.. _ceph-fs: https://opendev.org/openstack/charm-ceph-fs/
.. _ceph-iscsi: https://opendev.org/openstack/charm-ceph-iscsi/
.. _ceph-mon: https://opendev.org/openstack/charm-ceph-mon/
.. _ceph-osd: https://opendev.org/openstack/charm-ceph-osd/
.. _ceph-proxy: https://opendev.org/openstack/charm-ceph-proxy/
.. _ceph-radosgw: https://opendev.org/openstack/charm-ceph-radosgw/
.. _ceph-rbd-mirror: https://opendev.org/openstack/charm-ceph-rbd-mirror/
.. _cinder-purestorage: https://opendev.org/openstack/charm-cinder-purestorage/
.. _designate-bind: https://opendev.org/openstack/charm-designate-bind/
.. _glance-simplestreams-sync: https://opendev.org/openstack/charm-glance-simplestreams-sync/
.. _gnocchi: https://opendev.org/openstack/charm-gnocchi/
.. _hacluster: https://opendev.org/openstack/charm-hacluster/
.. _ovn-central: https://opendev.org/x/charm-ovn-central
.. _ovn-chassis: https://opendev.org/x/charm-ovn-chassis
.. _ovn-dedicated-chassis: https://opendev.org/x/charm-ovn-dedicated-chassis
.. _pacemaker-remote: https://opendev.org/openstack/charm-pacemaker-remote/
.. _percona-cluster: https://opendev.org/openstack/charm-percona-cluster/
.. _rabbitmq-server: https://opendev.org/openstack/charm-rabbitmq-server/
.. _trilio-data-mover: https://opendev.org/openstack/charm-trilio-data-mover/
.. _trilio-dm-api: https://opendev.org/openstack/charm-trilio-dm-api/
.. _trilio-horizon-plugin: https://opendev.org/openstack/charm-trilio-horizon-plugin/
.. _trilio-wlm: https://opendev.org/openstack/charm-trilio-wlm/
.. _vault: https://opendev.org/openstack/charm-vault/
