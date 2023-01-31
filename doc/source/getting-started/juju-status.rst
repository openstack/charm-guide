:orphan:

.. _juju_status:

===============
OpenStack cloud
===============

The below :command:`juju status --relations` output represents the cloud
installed from the instructions given on the :doc:`Deploy OpenStack
<deploy>` Getting Started page.

.. code-block:: console

   Model      Controller       Cloud/Region     Version  SLA          Timestamp
   openstack  maas-controller  mycloud/default  2.9.38   unsupported  21:01:48Z

   App                     Version  Status  Scale  Charm                   Channel        Rev  Exposed  Message
   ceph-mon                17.2.0   active      3  ceph-mon                quincy/stable  146  no       Unit is ready and clustered
   ceph-osd                17.2.0   active      3  ceph-osd                quincy/stable  541  no       Unit is ready (2 OSD)
   ceph-radosgw            17.2.0   active      1  ceph-radosgw            quincy/stable  542  no       Unit is ready
   cinder                  21.1.0   active      1  cinder                  zed/stable     581  no       Unit is ready
   cinder-ceph             21.1.0   active      1  cinder-ceph             zed/stable     513  no       Unit is ready
   cinder-mysql-router     8.0.32   active      1  mysql-router            8.0/stable      35  no       Unit is ready
   dashboard-mysql-router  8.0.32   active      1  mysql-router            8.0/stable      35  no       Unit is ready
   glance                  25.0.0   active      1  glance                  zed/stable     560  no       Unit is ready
   glance-mysql-router     8.0.32   active      1  mysql-router            8.0/stable      35  no       Unit is ready
   keystone                22.0.0   active      1  keystone                zed/stable     591  no       Application Ready
   keystone-mysql-router   8.0.32   active      1  mysql-router            8.0/stable      35  no       Unit is ready
   mysql-innodb-cluster    8.0.32   active      3  mysql-innodb-cluster    8.0/stable      39  no       Unit is ready: Mode: R/O, Cluster is ONLINE and can tolerate up to ONE failure.
   neutron-api             21.0.0   active      1  neutron-api             zed/stable     546  no       Unit is ready
   neutron-api-plugin-ovn  21.0.0   active      1  neutron-api-plugin-ovn  zed/stable      45  no       Unit is ready
   neutron-mysql-router    8.0.32   active      1  mysql-router            8.0/stable      35  no       Unit is ready
   nova-cloud-controller   26.1.0   active      1  nova-cloud-controller   zed/stable     633  no       Unit is ready
   nova-compute            26.1.0   active      3  nova-compute            zed/stable     626  no       Unit is ready
   nova-mysql-router       8.0.32   active      1  mysql-router            8.0/stable      35  no       Unit is ready
   openstack-dashboard     23.0.0   active      1  openstack-dashboard     zed/stable     564  no       Unit is ready
   ovn-central             22.09.0  active      3  ovn-central             22.09/stable    75  no       Unit is ready (leader: ovnnb_db, ovnsb_db)
   ovn-chassis             22.09.0  active      3  ovn-chassis             22.09/stable   109  no       Unit is ready
   placement               8.0.0    active      1  placement               zed/stable      67  no       Unit is ready
   placement-mysql-router  8.0.32   active      1  mysql-router            8.0/stable      35  no       Unit is ready
   rabbitmq-server         3.9.13   active      1  rabbitmq-server         3.9/stable     154  no       Unit is ready
   vault                   1.8.8    active      1  vault                   1.8/stable      86  no       Unit is ready (active: true, mlock: disabled)
   vault-mysql-router      8.0.32   active      1  mysql-router            8.0/stable      35  no       Unit is ready

   Unit                         Workload  Agent  Machine  Public address  Ports               Message
   ceph-mon/0*                  active    idle   0/lxd/0  10.246.114.12                       Unit is ready and clustered
   ceph-mon/1                   active    idle   1/lxd/0  10.246.114.24                       Unit is ready and clustered
   ceph-mon/2                   active    idle   2/lxd/0  10.246.114.38                       Unit is ready and clustered
   ceph-osd/0*                  active    idle   0        10.246.114.17                       Unit is ready (2 OSD)
   ceph-osd/1                   active    idle   1        10.246.114.7                        Unit is ready (4 OSD)
   ceph-osd/2                   active    idle   2        10.246.114.31                       Unit is ready (4 OSD)
   ceph-radosgw/0*              active    idle   0/lxd/1  10.246.114.11   80/tcp              Unit is ready
   cinder/0*                    active    idle   2        10.246.114.31   8776/tcp            Unit is ready
     cinder-ceph/0*             active    idle            10.246.114.31                       Unit is ready
     cinder-mysql-router/0*     active    idle            10.246.114.31                       Unit is ready
   glance/0*                    active    idle   2/lxd/1  10.246.114.20   9292/tcp            Unit is ready
     glance-mysql-router/0*     active    idle            10.246.114.20                       Unit is ready
   keystone/0*                  active    idle   0/lxd/2  10.246.114.14   5000/tcp            Unit is ready
     keystone-mysql-router/0*   active    idle            10.246.114.14                       Unit is ready
   mysql-innodb-cluster/0*      active    idle   0/lxd/3  10.246.114.15                       Unit is ready: Mode: R/O, Cluster is ONLINE and can tolerate up to ONE failure.
   mysql-innodb-cluster/1       active    idle   1/lxd/1  10.246.114.26                       Unit is ready: Mode: R/W, Cluster is ONLINE and can tolerate up to ONE failure.
   mysql-innodb-cluster/2       active    idle   2/lxd/2  10.246.114.19                       Unit is ready: Mode: R/O, Cluster is ONLINE and can tolerate up to ONE failure.
   neutron-api/0*               active    idle   1/lxd/2  10.246.114.37   9696/tcp            Unit is ready
     neutron-api-plugin-ovn/0*  active    idle            10.246.114.37                       Unit is ready
     neutron-mysql-router/0*    active    idle            10.246.114.37                       Unit is ready
   nova-cloud-controller/0*     active    idle   0/lxd/4  10.246.114.28   8774/tcp,8775/tcp   Unit is ready
     nova-mysql-router/0*       active    idle            10.246.114.28                       Unit is ready
   nova-compute/0*              active    idle   0        10.246.114.17                       Unit is ready
     ovn-chassis/0*             active    idle            10.246.114.17                       Unit is ready
   nova-compute/1               active    idle   1        10.246.114.7                        Unit is ready
     ovn-chassis/1              active    idle            10.246.114.7                        Unit is ready
   nova-compute/2               active    idle   2        10.246.114.31                       Unit is ready
     ovn-chassis/2              active    idle            10.246.114.31                       Unit is ready
   openstack-dashboard/0*       active    idle   1/lxd/3  10.246.114.52   80/tcp,443/tcp      Unit is ready
     dashboard-mysql-router/0*  active    idle            10.246.114.52                       Unit is ready
   ovn-central/0*               active    idle   0/lxd/5  10.246.114.29   6641/tcp,6642/tcp   Unit is ready (leader: ovnnb_db, ovnsb_db)
   ovn-central/1                active    idle   1/lxd/4  10.246.114.25   6641/tcp,6642/tcp   Unit is ready
   ovn-central/2                active    idle   2/lxd/3  10.246.114.39   6641/tcp,6642/tcp   Unit is ready (northd: active)
   placement/0*                 active    idle   2/lxd/4  10.246.114.22   8778/tcp            Unit is ready
     placement-mysql-router/0*  active    idle            10.246.114.22                       Unit is ready
   rabbitmq-server/0*           active    idle   2/lxd/5  10.246.114.21   5672/tcp,15672/tcp  Unit is ready
   vault/0*                     active    idle   1/lxd/5  10.246.114.51   8200/tcp            Unit is ready (active: true, mlock: disabled)
     vault-mysql-router/0*      active    idle            10.246.114.51                       Unit is ready

   Machine  State    Address        Inst id              Series  AZ       Message
   0        started  10.246.114.17  node-sparky          jammy   default  Deployed
   0/lxd/0  started  10.246.114.12  juju-b922cd-0-lxd-0  jammy   default  Container started
   0/lxd/1  started  10.246.114.11  juju-b922cd-0-lxd-1  jammy   default  Container started
   0/lxd/2  started  10.246.114.14  juju-b922cd-0-lxd-2  jammy   default  Container started
   0/lxd/3  started  10.246.114.15  juju-b922cd-0-lxd-3  jammy   default  Container started
   0/lxd/4  started  10.246.114.28  juju-b922cd-0-lxd-4  jammy   default  Container started
   0/lxd/5  started  10.246.114.29  juju-b922cd-0-lxd-5  jammy   default  Container started
   1        started  10.246.114.7   node-laveran         jammy   default  Deployed
   1/lxd/0  started  10.246.114.24  juju-b922cd-1-lxd-0  jammy   default  Container started
   1/lxd/1  started  10.246.114.26  juju-b922cd-1-lxd-1  jammy   default  Container started
   1/lxd/2  started  10.246.114.37  juju-b922cd-1-lxd-2  jammy   default  Container started
   1/lxd/3  started  10.246.114.52  juju-b922cd-1-lxd-3  jammy   default  Container started
   1/lxd/4  started  10.246.114.25  juju-b922cd-1-lxd-4  jammy   default  Container started
   1/lxd/5  started  10.246.114.51  juju-b922cd-1-lxd-5  jammy   default  Container started
   2        started  10.246.114.31  node-fontana         jammy   default  Deployed
   2/lxd/0  started  10.246.114.38  juju-b922cd-2-lxd-0  jammy   default  Container started
   2/lxd/1  started  10.246.114.20  juju-b922cd-2-lxd-1  jammy   default  Container started
   2/lxd/2  started  10.246.114.19  juju-b922cd-2-lxd-2  jammy   default  Container started
   2/lxd/3  started  10.246.114.39  juju-b922cd-2-lxd-3  jammy   default  Container started
   2/lxd/4  started  10.246.114.22  juju-b922cd-2-lxd-4  jammy   default  Container started
   2/lxd/5  started  10.246.114.21  juju-b922cd-2-lxd-5  jammy   default  Container started

   Relation provider                      Requirer                                     Interface                       Type         Message
   ceph-mon:client                        cinder-ceph:ceph                             ceph-client                     regular
   ceph-mon:client                        glance:ceph                                  ceph-client                     regular
   ceph-mon:client                        nova-compute:ceph                            ceph-client                     regular
   ceph-mon:mon                           ceph-mon:mon                                 ceph                            peer
   ceph-mon:osd                           ceph-osd:mon                                 ceph-osd                        regular
   ceph-mon:radosgw                       ceph-radosgw:mon                             ceph-radosgw                    regular
   ceph-radosgw:cluster                   ceph-radosgw:cluster                         swift-ha                        peer
   cinder-ceph:ceph-access                nova-compute:ceph-access                     cinder-ceph-key                 regular
   cinder-ceph:storage-backend            cinder:storage-backend                       cinder-backend                  subordinate
   cinder-mysql-router:shared-db          cinder:shared-db                             mysql-shared                    subordinate
   cinder:cinder-volume-service           nova-cloud-controller:cinder-volume-service  cinder                          regular
   cinder:cluster                         cinder:cluster                               cinder-ha                       peer
   dashboard-mysql-router:shared-db       openstack-dashboard:shared-db                mysql-shared                    subordinate
   glance-mysql-router:shared-db          glance:shared-db                             mysql-shared                    subordinate
   glance:cluster                         glance:cluster                               glance-ha                       peer
   glance:image-service                   cinder:image-service                         glance                          regular
   glance:image-service                   nova-cloud-controller:image-service          glance                          regular
   glance:image-service                   nova-compute:image-service                   glance                          regular
   keystone-mysql-router:shared-db        keystone:shared-db                           mysql-shared                    subordinate
   keystone:cluster                       keystone:cluster                             keystone-ha                     peer
   keystone:identity-service              ceph-radosgw:identity-service                keystone                        regular
   keystone:identity-service              cinder:identity-service                      keystone                        regular
   keystone:identity-service              glance:identity-service                      keystone                        regular
   keystone:identity-service              neutron-api:identity-service                 keystone                        regular
   keystone:identity-service              nova-cloud-controller:identity-service       keystone                        regular
   keystone:identity-service              openstack-dashboard:identity-service         keystone                        regular
   keystone:identity-service              placement:identity-service                   keystone                        regular
   mysql-innodb-cluster:cluster           mysql-innodb-cluster:cluster                 mysql-innodb-cluster            peer
   mysql-innodb-cluster:coordinator       mysql-innodb-cluster:coordinator             coordinator                     peer
   mysql-innodb-cluster:db-router         cinder-mysql-router:db-router                mysql-router                    regular
   mysql-innodb-cluster:db-router         dashboard-mysql-router:db-router             mysql-router                    regular
   mysql-innodb-cluster:db-router         glance-mysql-router:db-router                mysql-router                    regular
   mysql-innodb-cluster:db-router         keystone-mysql-router:db-router              mysql-router                    regular
   mysql-innodb-cluster:db-router         neutron-mysql-router:db-router               mysql-router                    regular
   mysql-innodb-cluster:db-router         nova-mysql-router:db-router                  mysql-router                    regular
   mysql-innodb-cluster:db-router         placement-mysql-router:db-router             mysql-router                    regular
   mysql-innodb-cluster:db-router         vault-mysql-router:db-router                 mysql-router                    regular
   neutron-api-plugin-ovn:neutron-plugin  neutron-api:neutron-plugin-api-subordinate   neutron-plugin-api-subordinate  subordinate
   neutron-api:cluster                    neutron-api:cluster                          neutron-api-ha                  peer
   neutron-api:neutron-api                nova-cloud-controller:neutron-api            neutron-api                     regular
   neutron-mysql-router:shared-db         neutron-api:shared-db                        mysql-shared                    subordinate
   nova-cloud-controller:cluster          nova-cloud-controller:cluster                nova-ha                         peer
   nova-compute:cloud-compute             nova-cloud-controller:cloud-compute          nova-compute                    regular
   nova-compute:compute-peer              nova-compute:compute-peer                    nova                            peer
   nova-mysql-router:shared-db            nova-cloud-controller:shared-db              mysql-shared                    subordinate
   openstack-dashboard:cluster            openstack-dashboard:cluster                  openstack-dashboard-ha          peer
   ovn-central:coordinator                ovn-central:coordinator                      coordinator                     peer
   ovn-central:ovsdb                      ovn-chassis:ovsdb                            ovsdb                           regular
   ovn-central:ovsdb-cms                  neutron-api-plugin-ovn:ovsdb-cms             ovsdb-cms                       regular
   ovn-central:ovsdb-peer                 ovn-central:ovsdb-peer                       ovsdb-cluster                   peer
   ovn-chassis:nova-compute               nova-compute:neutron-plugin                  neutron-plugin                  subordinate
   placement-mysql-router:shared-db       placement:shared-db                          mysql-shared                    subordinate
   placement:cluster                      placement:cluster                            openstack-ha                    peer
   placement:placement                    nova-cloud-controller:placement              placement                       regular
   rabbitmq-server:amqp                   cinder:amqp                                  rabbitmq                        regular
   rabbitmq-server:amqp                   glance:amqp                                  rabbitmq                        regular
   rabbitmq-server:amqp                   neutron-api:amqp                             rabbitmq                        regular
   rabbitmq-server:amqp                   nova-cloud-controller:amqp                   rabbitmq                        regular
   rabbitmq-server:amqp                   nova-compute:amqp                            rabbitmq                        regular
   rabbitmq-server:cluster                rabbitmq-server:cluster                      rabbitmq-ha                     peer
   vault-mysql-router:shared-db           vault:shared-db                              mysql-shared                    subordinate
   vault:certificates                     ceph-radosgw:certificates                    tls-certificates                regular
   vault:certificates                     cinder:certificates                          tls-certificates                regular
   vault:certificates                     glance:certificates                          tls-certificates                regular
   vault:certificates                     keystone:certificates                        tls-certificates                regular
   vault:certificates                     mysql-innodb-cluster:certificates            tls-certificates                regular
   vault:certificates                     neutron-api-plugin-ovn:certificates          tls-certificates                regular
   vault:certificates                     neutron-api:certificates                     tls-certificates                regular
   vault:certificates                     nova-cloud-controller:certificates           tls-certificates                regular
   vault:certificates                     openstack-dashboard:certificates             tls-certificates                regular
   vault:certificates                     ovn-central:certificates                     tls-certificates                regular
   vault:certificates                     ovn-chassis:certificates                     tls-certificates                regular
   vault:certificates                     placement:certificates                       tls-certificates                regular
   vault:cluster                          vault:cluster                                vault-ha                        peer
