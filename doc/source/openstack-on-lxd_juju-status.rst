:orphan:

.. _openstack_on_lxd_juju_status:

======================
OpenStack on LXD cloud
======================

The below :command:`juju status --relations` output represents the cloud
installed from the instructions given on the :doc:`OpenStack on LXD
<openstack-on-lxd>` page.

.. code-block:: console

   Model    Controller  Cloud/Region         Version  SLA          Timestamp
   default  lxd         localhost/localhost  2.8.0    unsupported  16:52:05Z

   App                    Version      Status   Scale  Charm                  Store       Rev  OS      Notes
   ceilometer             10.0.1       blocked      1  ceilometer             jujucharms  273  ubuntu
   ceilometer-agent       12.1.0       active       1  ceilometer-agent       jujucharms  263  ubuntu
   ceph-mon               13.2.8       active       3  ceph-mon               jujucharms   48  ubuntu
   ceph-osd               13.2.8       active       3  ceph-osd               jujucharms  303  ubuntu
   ceph-radosgw           13.2.8       active       1  ceph-radosgw           jujucharms  288  ubuntu
   cinder                 14.0.4       active       1  cinder                 jujucharms  303  ubuntu
   cinder-ceph            14.0.4       active       1  cinder-ceph            jujucharms  256  ubuntu
   designate              6.0.1        active       1  designate              jujucharms   44  ubuntu
   designate-bind         9.11.3+dfsg  active       1  designate-bind         jujucharms   29  ubuntu
   glance                 18.0.1       active       1  glance                 jujucharms  297  ubuntu
   gnocchi                4.3.2        active       1  gnocchi                jujucharms   40  ubuntu
   heat                   12.0.0       active       1  heat                   jujucharms  276  ubuntu
   keystone               15.0.0       active       1  keystone               jujucharms  315  ubuntu
   memcached                           active       1  memcached              jujucharms   29  ubuntu
   mysql                  5.7.20       active       1  percona-cluster        jujucharms  290  ubuntu
   neutron-api            14.1.0       active       1  neutron-api            jujucharms  286  ubuntu
   neutron-gateway        14.1.0       active       1  neutron-gateway        jujucharms  282  ubuntu
   neutron-openvswitch    14.1.0       active       1  neutron-openvswitch    jujucharms  276  ubuntu
   nova-cloud-controller  19.1.0       active       1  nova-cloud-controller  jujucharms  345  ubuntu
   nova-compute           19.1.0       active       1  nova-compute           jujucharms  317  ubuntu
   openstack-dashboard    15.2.0       active       1  openstack-dashboard    jujucharms  304  ubuntu
   rabbitmq-server        3.6.10       active       1  rabbitmq-server        jujucharms  102  ubuntu

   Unit                      Workload  Agent  Machine  Public address  Ports                       Message
   ceilometer/0*             active    idle   0        10.0.8.123                                  Unit is ready
   ceph-mon/0*               active    idle   1        10.0.8.68                                   Unit is ready and clustered
   ceph-mon/1                active    idle   2        10.0.8.23                                   Unit is ready and clustered
   ceph-mon/2                active    idle   3        10.0.8.17                                   Unit is ready and clustered
   ceph-osd/0                active    idle   4        10.0.8.4                                    Unit is ready (1 OSD)
   ceph-osd/1*               active    idle   5        10.0.8.107                                  Unit is ready (1 OSD)
   ceph-osd/2                active    idle   6        10.0.8.179                                  Unit is ready (1 OSD)
   ceph-radosgw/0*           active    idle   7        10.0.8.75       80/tcp                      Unit is ready
   cinder/0*                 active    idle   8        10.0.8.190      8776/tcp                    Unit is ready
     cinder-ceph/0*          active    idle            10.0.8.190                                  Unit is ready
   designate-bind/0*         active    idle   10       10.0.8.151                                  Unit is ready
   designate/0*              active    idle   9        10.0.8.156      9001/tcp                    Unit is ready
   glance/0*                 active    idle   11       10.0.8.163      9292/tcp                    Unit is ready
   gnocchi/0*                active    idle   12       10.0.8.112      8041/tcp                    Unit is ready
   heat/0*                   active    idle   13       10.0.8.133      8000/tcp,8004/tcp           Unit is ready
   keystone/0*               active    idle   14       10.0.8.170      5000/tcp                    Unit is ready
   memcached/0*              active    idle   15       10.0.8.103      11211/tcp                   Unit is ready
   mysql/0*                  active    idle   16       10.0.8.84       3306/tcp                    Unit is ready
   neutron-api/0*            active    idle   17       10.0.8.62       9696/tcp                    Unit is ready
   neutron-gateway/0*        active    idle   18       10.0.8.24                                   Unit is ready
   nova-cloud-controller/0*  active    idle   19       10.0.8.118      8774/tcp,8775/tcp,8778/tcp  Unit is ready
   nova-compute/0*           active    idle   20       10.0.8.92                                   Unit is ready
     ceilometer-agent/0*     active    idle            10.0.8.92                                   Unit is ready
     neutron-openvswitch/0*  active    idle            10.0.8.92                                   Unit is ready
   openstack-dashboard/0*    active    idle   21       10.0.8.69       80/tcp,443/tcp              Unit is ready
   rabbitmq-server/0*        active    idle   22       10.0.8.149      5672/tcp                    Unit is ready

   Machine  State    DNS         Inst id         Series  AZ  Message
   0        started  10.0.8.123  juju-472b85-0   bionic      Running
   1        started  10.0.8.68   juju-472b85-1   bionic      Running
   2        started  10.0.8.23   juju-472b85-2   bionic      Running
   3        started  10.0.8.17   juju-472b85-3   bionic      Running
   4        started  10.0.8.4    juju-472b85-4   bionic      Running
   5        started  10.0.8.107  juju-472b85-5   bionic      Running
   6        started  10.0.8.179  juju-472b85-6   bionic      Running
   7        started  10.0.8.75   juju-472b85-7   bionic      Running
   8        started  10.0.8.190  juju-472b85-8   bionic      Running
   9        started  10.0.8.156  juju-472b85-9   bionic      Running
   10       started  10.0.8.151  juju-472b85-10  bionic      Running
   11       started  10.0.8.163  juju-472b85-11  bionic      Running
   12       started  10.0.8.112  juju-472b85-12  bionic      Running
   13       started  10.0.8.133  juju-472b85-13  bionic      Running
   14       started  10.0.8.170  juju-472b85-14  bionic      Running
   15       started  10.0.8.103  juju-472b85-15  bionic      Running
   16       started  10.0.8.84   juju-472b85-16  bionic      Running
   17       started  10.0.8.62   juju-472b85-17  bionic      Running
   18       started  10.0.8.24   juju-472b85-18  bionic      Running
   19       started  10.0.8.118  juju-472b85-19  bionic      Running
   20       started  10.0.8.92   juju-472b85-20  bionic      Running
   21       started  10.0.8.69   juju-472b85-21  bionic      Running
   22       started  10.0.8.149  juju-472b85-22  bionic      Running

   Relation provider                        Requirer                                       Interface               Type         Message
   ceilometer-agent:nova-ceilometer         nova-compute:nova-ceilometer                   nova-ceilometer         subordinate
   ceilometer:ceilometer-service            ceilometer-agent:ceilometer-service            ceilometer              regular
   ceilometer:cluster                       ceilometer:cluster                             ceilometer-ha           peer
   ceph-mon:client                          cinder-ceph:ceph                               ceph-client             regular
   ceph-mon:client                          glance:ceph                                    ceph-client             regular
   ceph-mon:client                          gnocchi:storage-ceph                           ceph-client             regular
   ceph-mon:client                          nova-compute:ceph                              ceph-client             regular
   ceph-mon:mon                             ceph-mon:mon                                   ceph                    peer
   ceph-mon:osd                             ceph-osd:mon                                   ceph-osd                regular
   ceph-mon:radosgw                         ceph-radosgw:mon                               ceph-radosgw            regular
   ceph-radosgw:cluster                     ceph-radosgw:cluster                           swift-ha                peer
   cinder-ceph:ceph-access                  nova-compute:ceph-access                       cinder-ceph-key         regular
   cinder-ceph:storage-backend              cinder:storage-backend                         cinder-backend          subordinate
   cinder:cinder-volume-service             nova-cloud-controller:cinder-volume-service    cinder                  regular
   cinder:cluster                           cinder:cluster                                 cinder-ha               peer
   designate-bind:cluster                   designate-bind:cluster                         openstack-ha            peer
   designate-bind:dns-backend               designate:dns-backend                          bind-rndc               regular
   designate:cluster                        designate:cluster                              openstack-ha            peer
   glance:cluster                           glance:cluster                                 glance-ha               peer
   glance:image-service                     cinder:image-service                           glance                  regular
   glance:image-service                     nova-cloud-controller:image-service            glance                  regular
   glance:image-service                     nova-compute:image-service                     glance                  regular
   gnocchi:cluster                          gnocchi:cluster                                openstack-ha            peer
   gnocchi:metric-service                   ceilometer:metric-service                      gnocchi                 regular
   heat:cluster                             heat:cluster                                   heat-ha                 peer
   keystone:cluster                         keystone:cluster                               keystone-ha             peer
   keystone:identity-credentials            ceilometer:identity-credentials                keystone-credentials    regular
   keystone:identity-notifications          ceilometer:identity-notifications              keystone-notifications  regular
   keystone:identity-service                ceilometer:identity-service                    keystone                regular
   keystone:identity-service                ceph-radosgw:identity-service                  keystone                regular
   keystone:identity-service                cinder:identity-service                        keystone                regular
   keystone:identity-service                designate:identity-service                     keystone                regular
   keystone:identity-service                glance:identity-service                        keystone                regular
   keystone:identity-service                gnocchi:identity-service                       keystone                regular
   keystone:identity-service                heat:identity-service                          keystone                regular
   keystone:identity-service                neutron-api:identity-service                   keystone                regular
   keystone:identity-service                nova-cloud-controller:identity-service         keystone                regular
   keystone:identity-service                openstack-dashboard:identity-service           keystone                regular
   memcached:cache                          designate:coordinator-memcached                memcache                regular
   memcached:cache                          gnocchi:coordinator-memcached                  memcache                regular
   memcached:cluster                        memcached:cluster                              memcached-replication   peer
   mysql:cluster                            mysql:cluster                                  percona-cluster         peer
   mysql:shared-db                          cinder:shared-db                               mysql-shared            regular
   mysql:shared-db                          designate:shared-db                            mysql-shared            regular
   mysql:shared-db                          glance:shared-db                               mysql-shared            regular
   mysql:shared-db                          gnocchi:shared-db                              mysql-shared            regular
   mysql:shared-db                          heat:shared-db                                 mysql-shared            regular
   mysql:shared-db                          keystone:shared-db                             mysql-shared            regular
   mysql:shared-db                          neutron-api:shared-db                          mysql-shared            regular
   mysql:shared-db                          nova-cloud-controller:shared-db                mysql-shared            regular
   neutron-api:cluster                      neutron-api:cluster                            neutron-api-ha          peer
   neutron-api:neutron-api                  nova-cloud-controller:neutron-api              neutron-api             regular
   neutron-api:neutron-plugin-api           neutron-gateway:neutron-plugin-api             neutron-plugin-api      regular
   neutron-api:neutron-plugin-api           neutron-openvswitch:neutron-plugin-api         neutron-plugin-api      regular
   neutron-gateway:cluster                  neutron-gateway:cluster                        quantum-gateway-ha      peer
   neutron-gateway:quantum-network-service  nova-cloud-controller:quantum-network-service  quantum                 regular
   neutron-openvswitch:neutron-plugin       nova-compute:neutron-plugin                    neutron-plugin          subordinate
   nova-cloud-controller:cluster            nova-cloud-controller:cluster                  nova-ha                 peer
   nova-compute:cloud-compute               nova-cloud-controller:cloud-compute            nova-compute            regular
   nova-compute:compute-peer                nova-compute:compute-peer                      nova                    peer
   openstack-dashboard:cluster              openstack-dashboard:cluster                    openstack-dashboard-ha  peer
   rabbitmq-server:amqp                     ceilometer-agent:amqp                          rabbitmq                regular
   rabbitmq-server:amqp                     ceilometer:amqp                                rabbitmq                regular
   rabbitmq-server:amqp                     cinder:amqp                                    rabbitmq                regular
   rabbitmq-server:amqp                     designate:amqp                                 rabbitmq                regular
   rabbitmq-server:amqp                     glance:amqp                                    rabbitmq                regular
   rabbitmq-server:amqp                     gnocchi:amqp                                   rabbitmq                regular
   rabbitmq-server:amqp                     heat:amqp                                      rabbitmq                regular
   rabbitmq-server:amqp                     neutron-api:amqp                               rabbitmq                regular
   rabbitmq-server:amqp                     neutron-gateway:amqp                           rabbitmq                regular
   rabbitmq-server:amqp                     neutron-openvswitch:amqp                       rabbitmq                regular
   rabbitmq-server:amqp                     nova-cloud-controller:amqp                     rabbitmq                regular
   rabbitmq-server:amqp                     nova-compute:amqp                              rabbitmq                regular
   rabbitmq-server:cluster                  rabbitmq-server:cluster                        rabbitmq-ha             peer
