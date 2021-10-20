================
Supported charms
================

This page lists the charms that make up the OpenStack Charms project.

Each stable release of OpenStack Charms is backwards-compatible to cover all
currently-supported combinations of Ubuntu and OpenStack, relative to the
specific payload of each charm.

.. note::

   The latest stable charm revision should be used before proceeding with
   topological changes, charm application migrations, workload upgrades, series
   upgrades, or bug reports.

OpenStack charms (Stable)
-------------------------

These charms have stable releases with ongoing maintenance and testing. They
meet the requirements of payload project health, payload packaging,
upgradability, charm test gates, and the general release guidelines of the
OpenStack Charms project.

.. list-table::
   :header-rows: 1
   :widths: auto

   * - Charm
     - Initial release

   * - `aodh`_
     - < 16.10

   * - `barbican`_
     - 16.10

   * - `barbican-vault`_
     - 19.04

   * - `ceilometer`_
     - < 16.10

   * - `ceilometer-agent`_
     - < 16.10

   * - `cinder`_
     - < 16.10

   * - `cinder-backup`_
     - < 16.10

   * - `cinder-backup-swift-proxy`_
     - 20.05

   * - `cinder-ceph`_
     - < 16.10

   * - `designate`_
     - < 16.10

   * - `glance`_
     - < 16.10

   * - `heat`_
     - < 16.10

   * - `keystone`_
     - < 16.10

   * - `keystone-ldap`_
     - 17.02

   * - `keystone-saml-mellon`_
     - 20.05

   * - `manila`_
     - 20.02

   * - `manila-ganesha`_
     - 20.02

   * - `masakari`_
     - 20.05

   * - `masakari-monitors`_
     - 20.05

   * - `mysql-innodb-cluster`_
     - 20.05

   * - `mysql-router`_
     - 20.05

   * - `neutron-api`_
     - < 16.10

   * - `neutron-api-plugin-arista`_
     - 20.08

   * - `neutron-api-plugin-ovn`_
     - 20.05

   * - `neutron-dynamic-routing`_
     - 18.08

   * - `neutron-gateway`_
     - < 16.10

   * - `neutron-openvswitch`_
     - < 16.10

   * - `nova-cell-controller`_
     - 18.11

   * - `nova-cloud-controller`_
     - < 16.10

   * - `nova-compute`_
     - < 16.10

   * - `octavia`_
     - 19.04

   * - `octavia-dashboard`_
     - 19.04

   * - `octavia-diskimage-retrofit`_
     - 19.07

   * - `openstack-dashboard`_
     - < 16.10

   * - `placement`_
     - 19.10

   * - `swift-proxy`_
     - < 16.10

   * - `swift-storage`_
     - < 16.10

Supporting charms (Stable)
--------------------------

These charms have stable releases with ongoing maintenance and testing. They
are classified differently because the payload of each is not technically an
OpenStack project.

.. list-table::
   :header-rows: 1
   :widths: auto

   * - Charm
     - Initial release

   * - `ceph-fs`_
     - 20.02

   * - `ceph-iscsi`_
     - 20.10

   * - `ceph-mon`_
     - < 16.10

   * - `ceph-osd`_
     - < 16.10

   * - `ceph-proxy`_
     - 18.02

   * - `ceph-radosgw`_
     - < 16.10

   * - `ceph-rbd-mirror`_
     - 19.04

   * - `cinder-lvm`_
     - 21.10

   * - `cinder-netapp`_
     - 21.10

   * - `cinder-purestorage`_
     - 19.10

   * - `designate-bind`_
     - < 16.10

   * - `glance-simplestreams-sync`_
     - 18.05

   * - `gnocchi`_
     - 17.08

   * - `hacluster`_
     - < 16.10

   * - `ovn-central`_
     - 20.05

   * - `ovn-chassis`_
     - 20.05

   * - `ovn-dedicated-chassis`_
     - 20.05

   * - `pacemaker-remote`_
     - 20.05

   * - `percona-cluster`_
     - < 16.10

   * - `rabbitmq-server`_
     - < 16.10

   * - `trilio-data-mover`_
     - 20.08

   * - `trilio-dm-api`_
     - 20.08

   * - `trilio-horizon-plugin`_
     - 20.08

   * - `trilio-wlm`_
     - 20.08

   * - `vault`_
     - 18.05

Tech-preview charms (Beta)
--------------------------

These charms do not have stable releases, even though they may technically have
"stable/yy.mm" branches. Regardless of any maintenance and testing that these
charms may receive, some work (major bugs, payload packaging issues, project
issues, general QA) is still required before the charms are ready for
production use (promoted to Stable).

* `ceph-dashboard`_
* `ironic-api`_
* `ironic-conductor`_
* `keystone-kerberos`_
* `magnum`_
* `magnum-dashboard`_
* `manila-dashboard`_
* `manila-netapp`_
* `neutron-api-plugin-ironic`_
* `openstack-loadbalancer`_

Alpha charms (Edge)
-------------------

This classification of charms includes those which may be a proof-of-concept, a
test fixture, or one which is in active development. They are not intended to
be used in production. Supportability, upgradability, testability may be
lacking, either from a charm perspective, or from the workload package
perspective.

* `manila-generic`_
* `watcher`_
* `watcher-dashboard`_

Maintenance-mode charms
-----------------------

These charms are in maintenance mode, meaning that new features and new
releases are not actively being added or tested with them. Generally, these
were produced for a demo, PoC, or as an example.

* None at this time.

Deprecated charms
-----------------

These charms have reached EOL and are deprecated.

* `barbican-softhsm`_
* `ceph`_ - Use ceph-osd + ceph-mon instead.
* `glusterfs`_
* `manila-glusterfs`_
* `murano`_
* `neutron-api-odl`_
* `nova-compute-proxy`_
* `nova-lxd`_
* `odl-controller`_
* `openvswitch-odl`_
* `tempest`_
* `trove`_

.. LINKS
.. _aodh: https://opendev.org/openstack/charm-aodh/
.. _barbican: https://opendev.org/openstack/charm-barbican/
.. _barbican-vault: https://opendev.org/openstack/charm-barbican-vault/
.. _ceilometer: https://opendev.org/openstack/charm-ceilometer/
.. _ceilometer-agent: https://opendev.org/openstack/charm-ceilometer-agent/
.. _cinder: https://opendev.org/openstack/charm-cinder/
.. _cinder-backup: https://opendev.org/openstack/charm-cinder-backup/
.. _cinder-backup-swift-proxy: https://opendev.org/openstack/charm-cinder-backup-swift-proxy/
.. _cinder-ceph: https://opendev.org/openstack/charm-cinder-ceph/
.. _cinder-lvm: https://opendev.org/openstack/charm-cinder-lvm/
.. _cinder-netapp: https://opendev.org/openstack/charm-cinder-netapp/
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

.. _ceph-dashboard: https://opendev.org/openstack/charm-ceph-dashboard
.. _ironic-api: https://opendev.org/openstack/charm-ironic-api
.. _ironic-conductor: https://opendev.org/openstack/charm-ironic-conductor
.. _keystone-kerberos: https://opendev.org/openstack/charm-keystone-kerberos/
.. _magnum: https://opendev.org/openstack/charm-magnum
.. _magnum-dashboard: https://opendev.org/openstack/charm-magnum-dashboard
.. _manila-dashboard: https://opendev.org/openstack/charm-manila-dashboard
.. _manila-netapp: https://opendev.org/openstack/charm-manila-netapp
.. _neutron-api-plugin-ironic: https://opendev.org/openstack/charm-neutron-api-plugin-ironic
.. _openstack-loadbalancer: https://opendev.org/openstack/charm-openstack-loadbalancer

.. _manila-generic: https://opendev.org/openstack/charm-manila-generic/
.. _watcher: https://opendev.org/openstack/charm-watcher/
.. _watcher-dashboard: https://opendev.org/openstack/charm-watcher-dashboard/

.. _barbican-softhsm: https://opendev.org/openstack/charm-barbican-softhsm/
.. _ceph: https://opendev.org/openstack/charm-ceph/
.. _glusterfs: https://opendev.org/openstack/charm-glusterfs/
.. _manila-glusterfs: https://opendev.org/openstack/charm-manila-glusterfs/
.. _murano: https://opendev.org/openstack/charm-murano/
.. _neutron-api-odl: https://opendev.org/openstack/charm-neutron-api-odl/
.. _nova-compute-proxy: https://opendev.org/openstack/charm-nova-compute-proxy/
.. _nova-lxd: https://opendev.org/openstack/charm-nova-lxd/
.. _odl-controller: https://opendev.org/openstack/charm-odl-controller/
.. _openvswitch-odl: https://opendev.org/openstack/charm-openvswitch-odl/
.. _tempest: https://opendev.org/openstack/charm-tempest/
.. _trove: https://opendev.org/openstack/charm-trove/
