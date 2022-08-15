:orphan:

.. code-block:: console

   Model      Controller  Cloud/Region   Version  SLA          Timestamp
   openstack  controller  cloud/default  2.9.22   unsupported  02:25:29Z

   App                     Version  Status  Scale  Charm                   Store       Channel  Rev  OS      Message
   ceph-mon                16.2.6   active      3  ceph-mon                charmstore  stable    62  ubuntu  Unit is ready and clustered
   ceph-osd                16.2.6   active      3  ceph-osd                charmstore  stable   316  ubuntu  Unit is ready (1 OSD)
   ceph-radosgw            16.2.6   active      1  ceph-radosgw            charmstore  stable   300  ubuntu  Unit is ready
   cinder                  18.1.0   active      1  cinder                  charmstore  stable   319  ubuntu  Unit is ready
   cinder-ceph             18.1.0   active      1  cinder-ceph             charmstore  stable   268  ubuntu  Unit is ready
   cinder-mysql-router     8.0.28   active      1  mysql-router            charmstore  stable    15  ubuntu  Unit is ready
   dashboard-mysql-router  8.0.28   active      1  mysql-router            charmstore  stable    15  ubuntu  Unit is ready
   glance                  22.1.0   active      1  glance                  charmstore  stable   313  ubuntu  Unit is ready
   glance-mysql-router     8.0.28   active      1  mysql-router            charmstore  stable    15  ubuntu  Unit is ready
   keystone                19.0.0   active      3  keystone                charmstore  stable   330  ubuntu  Application Ready
   keystone-hacluster               active      3  hacluster               charmstore  stable    81  ubuntu  Unit is ready and clustered
   keystone-mysql-router   8.0.28   active      3  mysql-router            charmstore  stable    15  ubuntu  Unit is ready
   mysql-innodb-cluster    8.0.28   active      3  mysql-innodb-cluster    charmstore  stable    15  ubuntu  Unit is ready: Mode: R/O, Cluster is ONLINE and can tolerate up to ONE failure.
   neutron-api             18.1.1   active      1  neutron-api             charmstore  stable   304  ubuntu  Unit is ready
   neutron-api-plugin-ovn  18.1.1   active      1  neutron-api-plugin-ovn  charmstore  stable    50  ubuntu  Unit is ready
   neutron-mysql-router    8.0.28   active      1  mysql-router            charmstore  stable    15  ubuntu  Unit is ready
   nova-cloud-controller   23.1.0   active      1  nova-cloud-controller   charmstore  stable   363  ubuntu  Unit is ready
   nova-compute            23.1.0   active      3  nova-compute            charmstore  stable   337  ubuntu  Unit is ready
   nova-mysql-router       8.0.28   active      1  mysql-router            charmstore  stable    15  ubuntu  Unit is ready
   ntp                     3.5      active      3  ntp                     charmstore  stable    47  ubuntu  chrony: Ready
   openstack-dashboard     19.2.0   active      1  openstack-dashboard     charmstore  stable   318  ubuntu  Unit is ready
   ovn-central             20.12.0  active      3  ovn-central             charmstore  stable    16  ubuntu  Unit is ready (leader: ovnnb_db, ovnsb_db)
   ovn-chassis             20.12.0  active      3  ovn-chassis             charmstore  stable    21  ubuntu  Unit is ready
   placement               5.0.1    active      1  placement               charmstore  stable    32  ubuntu  Unit is ready
   placement-mysql-router  8.0.28   active      1  mysql-router            charmstore  stable    15  ubuntu  Unit is ready
   rabbitmq-server         3.8.2    active      1  rabbitmq-server         charmstore  stable   118  ubuntu  Unit is ready
   vault                   1.5.9    active      1  vault                   charmstore  stable    54  ubuntu  Unit is ready (active: true, mlock: disabled)
   vault-mysql-router      8.0.28   active      1  mysql-router            charmstore  stable    15  ubuntu  Unit is ready

   Unit                         Workload  Agent  Machine  Public address  Ports              Message
   ceph-mon/0*                  active    idle   0/lxd/0  10.246.114.55                      Unit is ready and clustered
   ceph-mon/1                   active    idle   1/lxd/0  10.246.114.56                      Unit is ready and clustered
   ceph-mon/2                   active    idle   2/lxd/0  10.246.114.35                      Unit is ready and clustered
   ceph-osd/0                   active    idle   0        10.246.114.21                      Unit is ready (1 OSD)
   ceph-osd/1                   active    idle   1        10.246.114.22                      Unit is ready (1 OSD)
   ceph-osd/2*                  active    idle   2        10.246.114.30                      Unit is ready (1 OSD)
   ceph-radosgw/0*              active    idle   0/lxd/1  10.246.114.47   80/tcp             Unit is ready
   cinder/0*                    active    idle   2        10.246.114.30   8776/tcp           Unit is ready
     cinder-ceph/0*             active    idle            10.246.114.30                      Unit is ready
     cinder-mysql-router/0*     active    idle            10.246.114.30                      Unit is ready
   glance/0*                    active    idle   2/lxd/1  10.246.114.34   9292/tcp           Unit is ready
     glance-mysql-router/0*     active    idle            10.246.114.34                      Unit is ready
   keystone/0*                  active    idle   0/lxd/2  10.246.114.58   5000/tcp           Unit is ready
     keystone-hacluster/0*      active    idle            10.246.114.58                      Unit is ready and clustered
     keystone-mysql-router/0*   active    idle            10.246.114.58                      Unit is ready
   keystone/1                   active    idle   1/lxd/6  10.246.114.37   5000/tcp           Unit is ready
     keystone-hacluster/1       active    idle            10.246.114.37                      Unit is ready and clustered
     keystone-mysql-router/1    active    idle            10.246.114.37                      Unit is ready
   keystone/2                   active    idle   2/lxd/6  10.246.114.38   5000/tcp           Unit is ready
     keystone-hacluster/2       active    idle            10.246.114.38                      Unit is ready and clustered
     keystone-mysql-router/2    active    idle            10.246.114.38                      Unit is ready
   mysql-innodb-cluster/0       active    idle   0/lxd/3  10.246.114.44                      Unit is ready: Mode: R/O, Cluster is ONLINE and can tolerate up to ONE failure.
   mysql-innodb-cluster/1*      active    idle   1/lxd/1  10.246.114.27                      Unit is ready: Mode: R/W, Cluster is ONLINE and can tolerate up to ONE failure.
   mysql-innodb-cluster/2       active    idle   2/lxd/2  10.246.114.32                      Unit is ready: Mode: R/O, Cluster is ONLINE and can tolerate up to ONE failure.
   neutron-api/0*               active    idle   1/lxd/2  10.246.114.57   9696/tcp           Unit is ready
     neutron-api-plugin-ovn/0*  active    idle            10.246.114.57                      Unit is ready
     neutron-mysql-router/0*    active    idle            10.246.114.57                      Unit is ready
   nova-cloud-controller/0*     active    idle   0/lxd/4  10.246.114.25   8774/tcp,8775/tcp  Unit is ready
     nova-mysql-router/0*       active    idle            10.246.114.25                      Unit is ready
   nova-compute/0               active    idle   0        10.246.114.21                      Unit is ready
     ntp/0*                     active    idle            10.246.114.21   123/udp            chrony: Ready
     ovn-chassis/0*             active    idle            10.246.114.21                      Unit is ready
   nova-compute/1               active    idle   1        10.246.114.22                      Unit is ready
     ntp/1                      active    idle            10.246.114.22   123/udp            chrony: Ready
     ovn-chassis/1              active    idle            10.246.114.22                      Unit is ready
   nova-compute/2*              active    idle   2        10.246.114.30                      Unit is ready
     ntp/2                      active    idle            10.246.114.30   123/udp            chrony: Ready
     ovn-chassis/2              active    idle            10.246.114.30                      Unit is ready
   openstack-dashboard/0*       active    idle   1/lxd/3  10.246.114.45   80/tcp,443/tcp     Unit is ready
     dashboard-mysql-router/0*  active    idle            10.246.114.45                      Unit is ready
   ovn-central/0*               active    idle   0/lxd/5  10.246.114.46   6641/tcp,6642/tcp  Unit is ready (leader: ovnnb_db, ovnsb_db)
   ovn-central/1                active    idle   1/lxd/4  10.246.114.24   6641/tcp,6642/tcp  Unit is ready (northd: active)
   ovn-central/2                active    idle   2/lxd/3  10.246.114.36   6641/tcp,6642/tcp  Unit is ready
   placement/0*                 active    idle   2/lxd/4  10.246.114.33   8778/tcp           Unit is ready
     placement-mysql-router/0*  active    idle            10.246.114.33                      Unit is ready
   rabbitmq-server/0*           active    idle   2/lxd/5  10.246.114.59   5672/tcp           Unit is ready
   vault/0*                     active    idle   1/lxd/5  10.246.114.26   8200/tcp           Unit is ready (active: true, mlock: disabled)
     vault-mysql-router/0*      active    idle            10.246.114.26                      Unit is ready

   Machine  State    DNS            Inst id              Series  AZ       Message
   0        started  10.246.114.21  node-fontana         focal   default  Deployed
   0/lxd/0  started  10.246.114.55  juju-8bef4d-0-lxd-0  focal   default  Container started
   0/lxd/1  started  10.246.114.47  juju-8bef4d-0-lxd-1  focal   default  Container started
   0/lxd/2  started  10.246.114.58  juju-8bef4d-0-lxd-2  focal   default  Container started
   0/lxd/3  started  10.246.114.44  juju-8bef4d-0-lxd-3  focal   default  Container started
   0/lxd/4  started  10.246.114.25  juju-8bef4d-0-lxd-4  focal   default  Container started
   0/lxd/5  started  10.246.114.46  juju-8bef4d-0-lxd-5  focal   default  Container started
   1        started  10.246.114.22  node-sarabhai        focal   default  Deployed
   1/lxd/0  started  10.246.114.56  juju-8bef4d-1-lxd-0  focal   default  Container started
   1/lxd/1  started  10.246.114.27  juju-8bef4d-1-lxd-1  focal   default  Container started
   1/lxd/2  started  10.246.114.57  juju-8bef4d-1-lxd-2  focal   default  Container started
   1/lxd/3  started  10.246.114.45  juju-8bef4d-1-lxd-3  focal   default  Container started
   1/lxd/4  started  10.246.114.24  juju-8bef4d-1-lxd-4  focal   default  Container started
   1/lxd/5  started  10.246.114.26  juju-8bef4d-1-lxd-5  focal   default  Container started
   1/lxd/6  started  10.246.114.37  juju-8bef4d-1-lxd-6  focal   default  Container started
   2        started  10.246.114.30  node-pytheas         focal   default  Deployed
   2/lxd/0  started  10.246.114.35  juju-8bef4d-2-lxd-0  focal   default  Container started
   2/lxd/1  started  10.246.114.34  juju-8bef4d-2-lxd-1  focal   default  Container started
   2/lxd/2  started  10.246.114.32  juju-8bef4d-2-lxd-2  focal   default  Container started
   2/lxd/3  started  10.246.114.36  juju-8bef4d-2-lxd-3  focal   default  Container started
   2/lxd/4  started  10.246.114.33  juju-8bef4d-2-lxd-4  focal   default  Container started
   2/lxd/5  started  10.246.114.59  juju-8bef4d-2-lxd-5  focal   default  Container started
   2/lxd/6  started  10.246.114.38  juju-8bef4d-2-lxd-6  focal   default  Container started

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
   keystone-hacluster:ha                  keystone:ha                                  hacluster                       subordinate
   keystone-hacluster:hanode              keystone-hacluster:hanode                    hacluster                       peer
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
   nova-compute:juju-info                 ntp:juju-info                                juju-info                       subordinate
   nova-mysql-router:shared-db            nova-cloud-controller:shared-db              mysql-shared                    subordinate
   ntp:ntp-peers                          ntp:ntp-peers                                ntp                             peer
   openstack-dashboard:cluster            openstack-dashboard:cluster                  openstack-dashboard-ha          peer
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
