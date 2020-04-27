.. _openstack-charms:

Charms
======

Each stable release of OpenStack Charms is backwards-compatible to cover all
currently-supported combinations of Ubuntu + OpenStack, relative to the
specific payload of each charm. The latest stable charm revision should be
used before proceeding with topological changes, charm application migrations,
workload upgrades, series upgrades, or bug reports.

OpenStack Charms (Stable)
~~~~~~~~~~~~~~~~~~~~~~~~~

These charms have stable releases with ongoing maintenance and testing. They
meet the requirements of payload project health, payload packaging,
upgradability, charm test gates, and the general release guidelines of
the OpenStack Charms project.

* `aodh <https://opendev.org/openstack/charm-aodh/>`_
* `barbican <https://opendev.org/openstack/charm-barbican/>`_
* `barbican-vault <https://opendev.org/openstack/charm-barbican-vault/>`_
* `ceilometer <https://opendev.org/openstack/charm-ceilometer/>`_
* `ceilometer-agent <https://opendev.org/openstack/charm-ceilometer-agent/>`_
* `cinder <https://opendev.org/openstack/charm-cinder/>`_
* `cinder-backup <https://opendev.org/openstack/charm-cinder-backup/>`_
* `cinder-ceph <https://opendev.org/openstack/charm-cinder-ceph/>`_
* `designate <https://opendev.org/openstack/charm-designate/>`_
* `glance <https://opendev.org/openstack/charm-glance/>`_
* `heat <https://opendev.org/openstack/charm-heat/>`_
* `keystone <https://opendev.org/openstack/charm-keystone/>`_
* `keystone-ldap <https://opendev.org/openstack/charm-keystone-ldap/>`_
* `keystone-saml-mellon <https://github.com/openstack-charmers/charm-keystone-saml-mellon/>`_
* `manila <https://opendev.org/openstack/charm-manila/>`_
* `manila-ganesha <https://opendev.org/openstack/charm-manila-ganesha/>`_
* `neutron-api <https://opendev.org/openstack/charm-neutron-api/>`_
* `neutron-dynamic-routing <https://opendev.org/openstack/charm-neutron-dynamic-routing/>`_
* `neutron-gateway <https://opendev.org/openstack/charm-neutron-gateway/>`_
* `neutron-openvswitch <https://opendev.org/openstack/charm-neutron-openvswitch/>`_
* `nova-cell-controller <https://opendev.org/openstack/charm-nova-cell-controller/>`_
* `nova-cloud-controller <https://opendev.org/openstack/charm-nova-cloud-controller/>`_
* `nova-compute <https://opendev.org/openstack/charm-nova-compute/>`_
* `octavia <https://opendev.org/openstack/charm-octavia/>`_
* `octavia-dashboard <https://opendev.org/openstack/charm-octavia-dashboard/>`_
* `octavia-diskimage-retrofit <https://opendev.org/openstack/charm-octavia-diskimage-retrofit/>`_
* `placement <https://opendev.org/openstack/charm-placement>`_
* `openstack-dashboard <https://opendev.org/openstack/charm-openstack-dashboard/>`_
* `swift-proxy <https://opendev.org/openstack/charm-swift-proxy/>`_
* `swift-storage <https://opendev.org/openstack/charm-swift-storage/>`_

Other Supporting Charms (Stable)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

These charms have stable releases with ongoing maintenance and testing.
They're classified differently because the payload of each is not technically
an OpenStack project.

* `ceph-fs <https://opendev.org/openstack/charm-ceph-fs/>`_
* `ceph-mon <https://opendev.org/openstack/charm-ceph-mon/>`_
* `ceph-osd <https://opendev.org/openstack/charm-ceph-osd/>`_
* `ceph-proxy <https://opendev.org/openstack/charm-ceph-proxy/>`_
* `ceph-radosgw <https://opendev.org/openstack/charm-ceph-radosgw/>`_
* `ceph-rbd-mirror <https://opendev.org/openstack/charm-ceph-rbd-mirror/>`_
* `designate-bind <https://opendev.org/openstack/charm-designate-bind/>`_
* `glance-simplestreams-sync <https://opendev.org/openstack/charm-glance-simplestreams-sync/>`_
* `gnocchi <https://opendev.org/openstack/charm-gnocchi/>`_
* `hacluster <https://opendev.org/openstack/charm-hacluster/>`_
* `percona-cluster <https://opendev.org/openstack/charm-percona-cluster/>`_
* `rabbitmq-server <https://opendev.org/openstack/charm-rabbitmq-server/>`_
* `vault <https://opendev.org/openstack/charm-vault/>`_

Tech-Preview Charms (Beta)
~~~~~~~~~~~~~~~~~~~~~~~~~~

While these charms may technically have "stable/yy.mm" branches, they are not
considered stable releases that are intended for production.

They may have ongoing maintenance and testing, but this classification
indicates that there may be one or more major blocking bugs, payload package
issues, or project issues that need to be addressed before the charm can be
considered for use in production.

* `masakari <https://opendev.org/openstack/charm-masakari/>`_
* `masakari-monitors <https://opendev.org/openstack/charm-masakari-monitors/>`_
* `mysql-innodb-cluster <https://opendev.org/openstack/charm-mysql-innodb-cluster>`_
* `mysql-router <https://opendev.org/openstack/charm-mysql-router>`_
* `neutron-api-plugin-ovn <https://opendev.org/openstack/charm-neutron-api-plugin-ovn>`_
* `ovn-central <https://opendev.org/x/charm-ovn-central>`_
* `ovn-chassis <https://opendev.org/x/charm-ovn-chassis>`_
* `ovn-dedicated-chassis <https://opendev.org/x/charm-ovn-dedicated-chassis>`_
* `pacemaker-remote <https://opendev.org/openstack/charm-pacemaker-remote/>`_

Alpha Charms (Edge)
~~~~~~~~~~~~~~~~~~~

This classification of charms includes those which may be a proof-of-concept,
a test fixture, or one which is in active development.  They are not intended
to be used in production.  Supportability, upgradability, testability may be
lacking, either from a charm perspective, or from the workload package
perspective.

* `barbican-softhsm <https://opendev.org/openstack/charm-barbican-softhsm/>`_
* `manila-generic <https://opendev.org/openstack/charm-manila-generic/>`_
* `tempest <https://opendev.org/openstack/charm-tempest/>`_


Maintenance-Mode Charms
~~~~~~~~~~~~~~~~~~~~~~~

These charms are in maintenance mode, meaning that new features and new
releases are not actively being added or tested with them. Generally, these
were produced for demo, PoC, or as examples.

* None at this time.


Deprecated Charms
~~~~~~~~~~~~~~~~~

These charms have reached EOL and are deprecated.

* `ceph <https://opendev.org/openstack/charm-ceph/>`_ - Use ceph-osd + ceph-mon instead.
* `glusterfs <https://opendev.org/openstack/charm-glusterfs/>`_
* `manila-glusterfs <https://opendev.org/openstack/charm-manila-glusterfs/>`_
* `murano <https://opendev.org/openstack/charm-murano/>`_
* `neutron-api-odl <https://opendev.org/openstack/charm-neutron-api-odl/>`_
* `nova-compute-proxy <https://opendev.org/openstack/charm-nova-compute-proxy/>`_
* `nova-lxd <https://opendev.org/openstack/charm-nova-lxd/>`_
* `odl-controller <https://opendev.org/openstack/charm-odl-controller/>`_
* `openvswitch-odl <https://opendev.org/openstack/charm-openvswitch-odl/>`_
* `trove <https://opendev.org/openstack/charm-trove/>`_
