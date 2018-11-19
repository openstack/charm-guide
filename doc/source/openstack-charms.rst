.. _openstack-charms:

Charms
======

OpenStack Charms
~~~~~~~~~~~~~~~~

These charms have stable releases with ongoing maintenance and testing.

* `aodh <https://git.openstack.org/cgit/openstack/charm-aodh/>`_
* `barbican <https://git.openstack.org/cgit/openstack/charm-barbican/>`_
* `ceilometer <https://git.openstack.org/cgit/openstack/charm-ceilometer/>`_
* `ceilometer-agent <https://git.openstack.org/cgit/openstack/charm-ceilometer-agent/>`_
* `cinder <https://git.openstack.org/cgit/openstack/charm-cinder/>`_
* `cinder-backup <https://git.openstack.org/cgit/openstack/charm-cinder-backup/>`_
* `cinder-ceph <https://git.openstack.org/cgit/openstack/charm-cinder-ceph/>`_
* `designate <https://git.openstack.org/cgit/openstack/charm-designate/>`_
* `glance <https://git.openstack.org/cgit/openstack/charm-glance/>`_
* `heat <https://git.openstack.org/cgit/openstack/charm-heat/>`_
* `keystone <https://git.openstack.org/cgit/openstack/charm-keystone/>`_
* `keystone-ldap <https://git.openstack.org/cgit/openstack/charm-keystone-ldap/>`_
* `neutron-api <https://git.openstack.org/cgit/openstack/charm-neutron-api/>`_
* `neutron-dynamic-routing <https://git.openstack.org/cgit/openstack/charm-neutron-dynamic-routing/>`_
* `neutron-gateway <https://git.openstack.org/cgit/openstack/charm-neutron-gateway/>`_
* `neutron-openvswitch <https://git.openstack.org/cgit/openstack/charm-neutron-openvswitch/>`_
* `nova-cell-controller <https://git.openstack.org/cgit/openstack/charm-nova-cell-controller/>`_
* `nova-cloud-controller <https://git.openstack.org/cgit/openstack/charm-nova-cloud-controller/>`_
* `nova-compute <https://git.openstack.org/cgit/openstack/charm-nova-compute/>`_
* `openstack-dashboard <https://git.openstack.org/cgit/openstack/charm-openstack-dashboard/>`_
* `swift-proxy <https://git.openstack.org/cgit/openstack/charm-swift-proxy/>`_
* `swift-storage <https://git.openstack.org/cgit/openstack/charm-swift-storage/>`_

Other Supporting Charms
~~~~~~~~~~~~~~~~~~~~~~~

These charms have stable releases with ongoing maintenance and testing.  They're
classified differently because the payload of each is not technically an OpenStack project.

* `percona-cluster <https://git.openstack.org/cgit/openstack/charm-percona-cluster/>`_
* `rabbitmq-server <https://git.openstack.org/cgit/openstack/charm-rabbitmq-server/>`_
* `lxd <https://git.openstack.org/cgit/openstack/charm-lxd/>`_
* `ceph-osd <https://git.openstack.org/cgit/openstack/charm-ceph-osd/>`_
* `ceph-mon <https://git.openstack.org/cgit/openstack/charm-ceph-mon/>`_
* `ceph-proxy <https://git.openstack.org/cgit/openstack/charm-ceph-proxy/>`_
* `ceph-radosgw <https://git.openstack.org/cgit/openstack/charm-ceph-radosgw/>`_
* `hacluster <https://git.openstack.org/cgit/openstack/charm-hacluster/>`_
* `designate-bind <https://git.openstack.org/cgit/openstack/charm-designate-bind/>`_
* `gnocchi <https://git.openstack.org/cgit/openstack/charm-gnocchi/>`_
* `vault <https://git.openstack.org/cgit/openstack/charm-vault/>`_

Development / Preview Charms
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

These charms are either in development or are released as a beta/preview.  This
is to indicate that additional validation, and further work may be necessary to
make a stable release for production use.

* `barbican-softhsm <https://git.openstack.org/cgit/openstack/charm-barbican-softhsm/>`_
* `barbican-vault <https://git.openstack.org/cgit/openstack/charm-barbican-vault/>`_
* `ceph-fs <https://git.openstack.org/cgit/openstack/charm-ceph-fs/>`_
* `cinder-backup <https://git.openstack.org/cgit/openstack/charm-cinder-backup/>`_
* `manila <https://git.openstack.org/cgit/openstack/charm-manila/>`_
* `manila-generic <https://git.openstack.org/cgit/openstack/charm-manila-generic/>`_
* `octavia <https://git.openstack.org/cgit/openstack/charm-octavia/>`_
* `tempest <https://git.openstack.org/cgit/openstack/charm-tempest/>`_

Maintenance-Mode Charms
~~~~~~~~~~~~~~~~~~~~~~~

These charms are in maintenance mode, meaning that new features and new releases
are not actively being added or tested with them.  Generally, these were produced
for demo, PoC, or as examples.

* None at this time.

Deprecated Charms
~~~~~~~~~~~~~~~~~

These charms have reached EOL and are deprecated.

* `ceph <https://git.openstack.org/cgit/openstack/charm-ceph/>`_ - Use ceph-osd + ceph-mon instead.
* `neutron-api-odl <https://git.openstack.org/cgit/openstack/charm-neutron-api-odl/>`_
* `openvswitch-odl <https://git.openstack.org/cgit/openstack/charm-openvswitch-odl/>`_
* `odl-controller <https://git.openstack.org/cgit/openstack/charm-odl-controller/>`_
