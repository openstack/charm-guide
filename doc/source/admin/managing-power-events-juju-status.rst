:orphan:

.. _reference_cloud:

Reference cloud
===============

.. note::

    The information on this page is associated with the topic of :ref:`Managing
    Power Events <managing_power_events>`. See that page for background
    information.

The cloud is represented in the form of ``juju status`` output.

.. code::

    Model      Controller        Cloud/Region  Version  SLA          Timestamp
    openstack  foundations-maas  maas_cloud    2.6.2    unsupported  16:26:29Z

    App                            Version       Status       Scale  Charm                          Store       Rev  OS      Notes
    aodh                           7.0.0         active           3  aodh                           jujucharms   83  ubuntu
    bcache-tuning                                active           9  bcache-tuning                  jujucharms   10  ubuntu
    canonical-livepatch                          active          22  canonical-livepatch            jujucharms   32  ubuntu
    ceilometer                     11.0.1        blocked          3  ceilometer                     jujucharms  339  ubuntu
    ceilometer-agent               11.0.1        active           7  ceilometer-agent               jujucharms  302  ubuntu
    ceph-mon                       13.2.4+dfsg1  active           3  ceph-mon                       jujucharms  390  ubuntu
    ceph-osd                       13.2.4+dfsg1  active           9  ceph-osd                       jujucharms  411  ubuntu
    ceph-radosgw                   13.2.4+dfsg1  active           3  ceph-radosgw                   jujucharms  334  ubuntu
    cinder                         13.0.3        active           3  cinder                         jujucharms  375  ubuntu
    cinder-ceph                    13.0.3        active           3  cinder-ceph                    jujucharms  300  ubuntu
    designate                      7.0.0         active           3  designate                      jujucharms  122  ubuntu
    designate-bind                 9.11.3+dfsg   active           2  designate-bind                 jujucharms   65  ubuntu
    elasticsearch                                active           2  elasticsearch                  jujucharms   37  ubuntu
    filebeat                       5.6.16        active          74  filebeat                       jujucharms   24  ubuntu
    glance                         17.0.0        active           3  glance                         jujucharms  372  ubuntu
    gnocchi                        4.3.2         active           3  gnocchi                        jujucharms   60  ubuntu
    grafana                                      active           1  grafana                        jujucharms   29  ubuntu
    graylog                        2.5.1         active           1  graylog                        jujucharms   31  ubuntu
    graylog-mongodb                3.6.3         active           1  mongodb                        jujucharms   52  ubuntu
    hacluster-aodh                               active           3  hacluster                      jujucharms  102  ubuntu
    hacluster-ceilometer                         active           3  hacluster                      jujucharms  102  ubuntu
    hacluster-cinder                             active           3  hacluster                      jujucharms  102  ubuntu
    hacluster-designate                          active           3  hacluster                      jujucharms  102  ubuntu
    hacluster-glance                             active           3  hacluster                      jujucharms  102  ubuntu
    hacluster-gnocchi                            active           3  hacluster                      jujucharms  102  ubuntu
    hacluster-heat                               active           3  hacluster                      jujucharms  102  ubuntu
    hacluster-horizon                            active           3  hacluster                      jujucharms  102  ubuntu
    hacluster-keystone                           active           3  hacluster                      jujucharms  102  ubuntu
    hacluster-mysql                              active           3  hacluster                      jujucharms  102  ubuntu
    hacluster-neutron                            active           3  hacluster                      jujucharms  102  ubuntu
    hacluster-nova                               active           3  hacluster                      jujucharms  102  ubuntu
    hacluster-radosgw                            active           3  hacluster                      jujucharms  102  ubuntu
    heat                           11.0.0        active           3  heat                           jujucharms  326  ubuntu
    keystone                       14.0.1        active           3  keystone                       jujucharms  445  ubuntu
    keystone-ldap                  14.0.1        active           3  keystone-ldap                  jujucharms   17  ubuntu
    landscape-haproxy                            unknown          1  haproxy                        jujucharms   50  ubuntu
    landscape-postgresql           10.8          maintenance      2  postgresql                     jujucharms  199  ubuntu
    landscape-rabbitmq-server      3.6.10        active           3  rabbitmq-server                jujucharms   89  ubuntu
    landscape-server                             active           3  landscape-server               jujucharms   32  ubuntu
    lldpd                                        active           9  lldpd                          jujucharms    5  ubuntu
    memcached                                    active           2  memcached                      jujucharms   23  ubuntu
    mysql                          5.7.20-29.24  active           3  percona-cluster                jujucharms  340  ubuntu
    nagios                                       active           1  nagios                         jujucharms   32  ubuntu
    neutron-api                    13.0.2        active           3  neutron-api                    jujucharms  401  ubuntu
    neutron-gateway                13.0.2        active           2  neutron-gateway                jujucharms  371  ubuntu
    neutron-openvswitch            13.0.2        active           7  neutron-openvswitch            jujucharms  358  ubuntu
    nova-cloud-controller          18.1.0        active           3  nova-cloud-controller          jujucharms  424  ubuntu
    nova-compute-kvm               18.1.0        active           5  nova-compute                   jujucharms  448  ubuntu
    nova-compute-lxd               18.1.0        active           2  nova-compute                   jujucharms  448  ubuntu
    nrpe-container                               active          51  nrpe                           jujucharms   57  ubuntu
    nrpe-host                                    active          32  nrpe                           jujucharms   57  ubuntu
    ntp                            3.2           active          24  ntp                            jujucharms   32  ubuntu
    openstack-dashboard            14.0.2        active           3  openstack-dashboard            jujucharms  425  ubuntu
    openstack-service-checks                     active           1  openstack-service-checks       jujucharms   18  ubuntu
    prometheus                                   active           1  prometheus2                    jujucharms   10  ubuntu
    prometheus-ceph-exporter                     active           1  prometheus-ceph-exporter       jujucharms    5  ubuntu
    prometheus-openstack-exporter                active           1  prometheus-openstack-exporter  jujucharms    7  ubuntu
    rabbitmq-server                3.6.10        active           3  rabbitmq-server                jujucharms  344  ubuntu
    telegraf                                     active          74  telegraf                       jujucharms   29  ubuntu
    telegraf-prometheus                          active           1  telegraf                       jujucharms   29  ubuntu
    thruk-agent                                  unknown          1  thruk-agent                    jujucharms    6  ubuntu

    Unit                              Workload     Agent  Machine   Public address  Ports                                    Message
    aodh/0*                           active       idle   18/lxd/0  10.244.40.236   8042/tcp                                 Unit is ready
      filebeat/46                     active       idle             10.244.40.236                                            Filebeat ready
      hacluster-aodh/0*               active       idle             10.244.40.236                                            Unit is ready and clustered
      nrpe-container/24               active       idle             10.244.40.236   icmp,5666/tcp                            ready
      telegraf/46                     active       idle             10.244.40.236   9103/tcp                                 Monitoring aodh/0
    aodh/1                            active       idle   20/lxd/0  10.244.41.74    8042/tcp                                 Unit is ready
      filebeat/61                     active       idle             10.244.41.74                                             Filebeat ready
      hacluster-aodh/1                active       idle             10.244.41.74                                             Unit is ready and clustered
      nrpe-container/38               active       idle             10.244.41.74    icmp,5666/tcp                            ready
      telegraf/61                     active       idle             10.244.41.74    9103/tcp                                 Monitoring aodh/1
    aodh/2                            active       idle   21/lxd/0  10.244.41.66    8042/tcp                                 Unit is ready
      filebeat/65                     active       idle             10.244.41.66                                             Filebeat ready
      hacluster-aodh/2                active       idle             10.244.41.66                                             Unit is ready and clustered
      nrpe-container/42               active       idle             10.244.41.66    icmp,5666/tcp                            ready
      telegraf/65                     active       idle             10.244.41.66    9103/tcp                                 Monitoring aodh/2
    ceilometer/0                      blocked      idle   18/lxd/1  10.244.40.239                                            Run the ceilometer-upgrade action on the leader to initialize ceilometer and gnocchi
      filebeat/51                     active       idle             10.244.40.239                                            Filebeat ready
      hacluster-ceilometer/1          active       idle             10.244.40.239                                            Unit is ready and clustered
      nrpe-container/28               active       idle             10.244.40.239   icmp,5666/tcp                            ready
      telegraf/51                     active       idle             10.244.40.239   9103/tcp                                 Monitoring ceilometer/0
    ceilometer/1                      blocked      idle   20/lxd/1  10.244.41.77                                             Run the ceilometer-upgrade action on the leader to initialize ceilometer and gnocchi
      filebeat/70                     active       idle             10.244.41.77                                             Filebeat ready
      hacluster-ceilometer/2          active       idle             10.244.41.77                                             Unit is ready and clustered
      nrpe-container/47               active       idle             10.244.41.77    icmp,5666/tcp                            ready
      telegraf/70                     active       idle             10.244.41.77    9103/tcp                                 Monitoring ceilometer/1
    ceilometer/2*                     blocked      idle   21/lxd/1  10.244.40.229                                            Run the ceilometer-upgrade action on the leader to initialize ceilometer and gnocchi
      filebeat/22                     active       idle             10.244.40.229                                            Filebeat ready
      hacluster-ceilometer/0*         active       idle             10.244.40.229                                            Unit is ready and clustered
      nrpe-container/4                active       idle             10.244.40.229   icmp,5666/tcp                            ready
      telegraf/22                     active       idle             10.244.40.229   9103/tcp                                 Monitoring ceilometer/2
    ceph-mon/0*                       active       idle   15/lxd/0  10.244.40.227                                            Unit is ready and clustered
      filebeat/17                     active       idle             10.244.40.227                                            Filebeat ready
      nrpe-container/2                active       idle             10.244.40.227   icmp,5666/tcp                            ready
      telegraf/17                     active       idle             10.244.40.227   9103/tcp                                 Monitoring ceph-mon/0
    ceph-mon/1                        active       idle   16/lxd/0  10.244.40.253                                            Unit is ready and clustered
      filebeat/47                     active       idle             10.244.40.253                                            Filebeat ready
      nrpe-container/25               active       idle             10.244.40.253   icmp,5666/tcp                            ready
      telegraf/47                     active       idle             10.244.40.253   9103/tcp                                 Monitoring ceph-mon/1
    ceph-mon/2                        active       idle   17/lxd/0  10.244.41.78                                             Unit is ready and clustered
      filebeat/71                     active       idle             10.244.41.78                                             Filebeat ready
      nrpe-container/48               active       idle             10.244.41.78    icmp,5666/tcp                            ready
      telegraf/71                     active       idle             10.244.41.78    9103/tcp                                 Monitoring ceph-mon/2
    ceph-osd/0*                       active       idle   15        10.244.40.206                                            Unit is ready (1 OSD)
      bcache-tuning/1                 active       idle             10.244.40.206                                            bcache devices tuned
      nrpe-host/16                    active       idle             10.244.40.206   icmp,5666/tcp                            ready
    ceph-osd/1                        active       idle   16        10.244.40.213                                            Unit is ready (1 OSD)
      bcache-tuning/8                 active       idle             10.244.40.213                                            bcache devices tuned
      nrpe-host/30                    active       idle             10.244.40.213   icmp,5666/tcp                            ready
    ceph-osd/2                        active       idle   17        10.244.40.220                                            Unit is ready (1 OSD)
      bcache-tuning/4                 active       idle             10.244.40.220                                            bcache devices tuned
      nrpe-host/23                    active       idle             10.244.40.220                                            ready
    ceph-osd/3                        active       idle   18        10.244.40.225                                            Unit is ready (1 OSD)
      bcache-tuning/5                 active       idle             10.244.40.225                                            bcache devices tuned
      nrpe-host/25                    active       idle             10.244.40.225   icmp,5666/tcp                            ready
    ceph-osd/4                        active       idle   19        10.244.40.221                                            Unit is ready (1 OSD)
      bcache-tuning/2                 active       idle             10.244.40.221                                            bcache devices tuned
      nrpe-host/18                    active       idle             10.244.40.221   icmp,5666/tcp                            ready
    ceph-osd/5                        active       idle   20        10.244.40.224                                            Unit is ready (1 OSD)
      bcache-tuning/6                 active       idle             10.244.40.224                                            bcache devices tuned
      nrpe-host/27                    active       idle             10.244.40.224   icmp,5666/tcp                            ready
    ceph-osd/6                        active       idle   21        10.244.40.222                                            Unit is ready (1 OSD)
      bcache-tuning/7                 active       idle             10.244.40.222                                            bcache devices tuned
      nrpe-host/29                    active       idle             10.244.40.222                                            ready
    ceph-osd/7                        active       idle   22        10.244.40.223                                            Unit is ready (1 OSD)
      bcache-tuning/3                 active       idle             10.244.40.223                                            bcache devices tuned
      nrpe-host/20                    active       idle             10.244.40.223   icmp,5666/tcp                            ready
    ceph-osd/8                        active       idle   23        10.244.40.219                                            Unit is ready (1 OSD)
      bcache-tuning/0*                active       idle             10.244.40.219                                            bcache devices tuned
      nrpe-host/14                    active       idle             10.244.40.219                                            ready
    ceph-radosgw/0*                   active       idle   15/lxd/1  10.244.40.228   80/tcp                                   Unit is ready
      filebeat/15                     active       idle             10.244.40.228                                            Filebeat ready
      hacluster-radosgw/0*            active       idle             10.244.40.228                                            Unit is ready and clustered
      nrpe-container/1                active       idle             10.244.40.228   icmp,5666/tcp                            ready
      telegraf/15                     active       idle             10.244.40.228   9103/tcp                                 Monitoring ceph-radosgw/0
    ceph-radosgw/1                    active       idle   16/lxd/1  10.244.40.241   80/tcp                                   Unit is ready
      filebeat/35                     active       idle             10.244.40.241                                            Filebeat ready
      hacluster-radosgw/2             active       idle             10.244.40.241                                            Unit is ready and clustered
      nrpe-container/15               active       idle             10.244.40.241   icmp,5666/tcp                            ready
      telegraf/35                     active       idle             10.244.40.241   9103/tcp                                 Monitoring ceph-radosgw/1
    ceph-radosgw/2                    active       idle   17/lxd/1  10.244.40.233   80/tcp                                   Unit is ready
      filebeat/21                     active       idle             10.244.40.233                                            Filebeat ready
      hacluster-radosgw/1             active       idle             10.244.40.233                                            Unit is ready and clustered
      nrpe-container/3                active       idle             10.244.40.233   icmp,5666/tcp                            ready
      telegraf/21                     active       idle             10.244.40.233   9103/tcp                                 Monitoring ceph-radosgw/2
    cinder/0*                         active       idle   15/lxd/2  10.244.40.249   8776/tcp                                 Unit is ready
      cinder-ceph/0*                  active       idle             10.244.40.249                                            Unit is ready
      filebeat/29                     active       idle             10.244.40.249                                            Filebeat ready
      hacluster-cinder/0*             active       idle             10.244.40.249                                            Unit is ready and clustered
      nrpe-container/9                active       idle             10.244.40.249   icmp,5666/tcp                            ready
      telegraf/29                     active       idle             10.244.40.249   9103/tcp                                 Monitoring cinder/0
    cinder/1                          active       idle   16/lxd/2  10.244.40.248   8776/tcp                                 Unit is ready
      cinder-ceph/2                   active       idle             10.244.40.248                                            Unit is ready
      filebeat/59                     active       idle             10.244.40.248                                            Filebeat ready
      hacluster-cinder/2              active       idle             10.244.40.248                                            Unit is ready and clustered
      nrpe-container/36               active       idle             10.244.40.248   icmp,5666/tcp                            ready
      telegraf/59                     active       idle             10.244.40.248   9103/tcp                                 Monitoring cinder/1
    cinder/2                          active       idle   17/lxd/2  10.244.41.2     8776/tcp                                 Unit is ready
      cinder-ceph/1                   active       idle             10.244.41.2                                              Unit is ready
      filebeat/42                     active       idle             10.244.41.2                                              Filebeat ready
      hacluster-cinder/1              active       idle             10.244.41.2                                              Unit is ready and clustered
      nrpe-container/21               active       idle             10.244.41.2     icmp,5666/tcp                            ready
      telegraf/42                     active       idle             10.244.41.2     9103/tcp                                 Monitoring cinder/2
    designate-bind/0*                 active       idle   16/lxd/3  10.244.40.250                                            Unit is ready
      filebeat/45                     active       idle             10.244.40.250                                            Filebeat ready
      nrpe-container/23               active       idle             10.244.40.250   icmp,5666/tcp                            ready
      telegraf/45                     active       idle             10.244.40.250   9103/tcp                                 Monitoring designate-bind/0
    designate-bind/1                  active       idle   17/lxd/3  10.244.40.255                                            Unit is ready
      filebeat/40                     active       idle             10.244.40.255                                            Filebeat ready
      nrpe-container/20               active       idle             10.244.40.255   icmp,5666/tcp                            ready
      telegraf/40                     active       idle             10.244.40.255   9103/tcp                                 Monitoring designate-bind/1
    designate/0*                      active       idle   18/lxd/2  10.244.41.70    9001/tcp                                 Unit is ready
      filebeat/57                     active       idle             10.244.41.70                                             Filebeat ready
      hacluster-designate/0*          active       idle             10.244.41.70                                             Unit is ready and clustered
      nrpe-container/34               active       idle             10.244.41.70    icmp,5666/tcp                            ready
      telegraf/57                     active       idle             10.244.41.70    9103/tcp                                 Monitoring designate/0
    designate/1                       active       idle   20/lxd/2  10.244.41.72    9001/tcp                                 Unit is ready
      filebeat/63                     active       idle             10.244.41.72                                             Filebeat ready
      hacluster-designate/1           active       idle             10.244.41.72                                             Unit is ready and clustered
      nrpe-container/40               active       idle             10.244.41.72    icmp,5666/tcp                            ready
      telegraf/63                     active       idle             10.244.41.72    9103/tcp                                 Monitoring designate/1
    designate/2                       active       idle   21/lxd/2  10.244.41.71    9001/tcp                                 Unit is ready
      filebeat/69                     active       idle             10.244.41.71                                             Filebeat ready
      hacluster-designate/2           active       idle             10.244.41.71                                             Unit is ready and clustered
      nrpe-container/46               active       idle             10.244.41.71    icmp,5666/tcp                            ready
      telegraf/69                     active       idle             10.244.41.71    9103/tcp                                 Monitoring designate/2
    elasticsearch/0                   active       idle   5         10.244.40.217   9200/tcp                                 Unit is ready
      canonical-livepatch/3           active       idle             10.244.40.217                                            Running kernel 4.15.0-50.54-generic, patchState: nothing-to-apply
      filebeat/4                      active       idle             10.244.40.217                                            Filebeat ready
      nrpe-host/3                     active       idle             10.244.40.217   icmp,5666/tcp                            ready
      ntp/4                           active       idle             10.244.40.217   123/udp                                  chrony: Ready
      telegraf/4                      active       idle             10.244.40.217   9103/tcp                                 Monitoring elasticsearch/0
    elasticsearch/1*                  active       idle   13        10.244.40.209   9200/tcp                                 Unit is ready
      canonical-livepatch/2           active       idle             10.244.40.209                                            Running kernel 4.15.0-50.54-generic, patchState: nothing-to-apply
      filebeat/3                      active       idle             10.244.40.209                                            Filebeat ready
      nrpe-host/2                     active       idle             10.244.40.209   icmp,5666/tcp                            ready
      ntp/3                           active       idle             10.244.40.209   123/udp                                  chrony: Ready
      telegraf/3                      active       idle             10.244.40.209   9103/tcp                                 Monitoring elasticsearch/1
    glance/0                          active       idle   15/lxd/3  10.244.40.237   9292/tcp                                 Unit is ready
      filebeat/36                     active       idle             10.244.40.237                                            Filebeat ready
      hacluster-glance/0*             active       idle             10.244.40.237                                            Unit is ready and clustered
      nrpe-container/16               active       idle             10.244.40.237   icmp,5666/tcp                            ready
      telegraf/36                     active       idle             10.244.40.237   9103/tcp                                 Monitoring glance/0
    glance/1                          active       idle   16/lxd/4  10.244.41.5     9292/tcp                                 Unit is ready
      filebeat/67                     active       idle             10.244.41.5                                              Filebeat ready
      hacluster-glance/2              active       idle             10.244.41.5                                              Unit is ready and clustered
      nrpe-container/44               active       idle             10.244.41.5     icmp,5666/tcp                            ready
      telegraf/66                     active       idle             10.244.41.5     9103/tcp                                 Monitoring glance/1
    glance/2*                         active       idle   17/lxd/4  10.244.40.234   9292/tcp                                 Unit is ready
      filebeat/37                     active       idle             10.244.40.234                                            Filebeat ready
      hacluster-glance/1              active       idle             10.244.40.234                                            Unit is ready and clustered
      nrpe-container/17               active       idle             10.244.40.234   icmp,5666/tcp                            ready
      telegraf/37                     active       idle             10.244.40.234   9103/tcp                                 Monitoring glance/2
    gnocchi/0                         active       idle   18/lxd/3  10.244.40.231   8041/tcp                                 Unit is ready
      filebeat/24                     active       idle             10.244.40.231                                            Filebeat ready
      hacluster-gnocchi/0*            active       idle             10.244.40.231                                            Unit is ready and clustered
      nrpe-container/5                active       idle             10.244.40.231   icmp,5666/tcp                            ready
      telegraf/24                     active       idle             10.244.40.231   9103/tcp                                 Monitoring gnocchi/0
    gnocchi/1                         active       idle   20/lxd/3  10.244.40.244   8041/tcp                                 Unit is ready
      filebeat/55                     active       idle             10.244.40.244                                            Filebeat ready
      hacluster-gnocchi/2             active       idle             10.244.40.244                                            Unit is ready and clustered
      nrpe-container/32               active       idle             10.244.40.244   icmp,5666/tcp                            ready
      telegraf/55                     active       idle             10.244.40.244   9103/tcp                                 Monitoring gnocchi/1
    gnocchi/2*                        active       idle   21/lxd/3  10.244.40.230   8041/tcp                                 Unit is ready
      filebeat/27                     active       idle             10.244.40.230                                            Filebeat ready
      hacluster-gnocchi/1             active       idle             10.244.40.230                                            Unit is ready and clustered
      nrpe-container/7                active       idle             10.244.40.230   icmp,5666/tcp                            ready
      telegraf/27                     active       idle             10.244.40.230   9103/tcp                                 Monitoring gnocchi/2
    grafana/0*                        active       idle   1         10.244.40.202   3000/tcp                                 Started snap.grafana.grafana
      canonical-livepatch/1           active       idle             10.244.40.202                                            Running kernel 4.15.0-50.54-generic, patchState: nothing-to-apply
      filebeat/2                      active       idle             10.244.40.202                                            Filebeat ready
      nrpe-host/1                     active       idle             10.244.40.202   icmp,5666/tcp                            ready
      ntp/2                           active       idle             10.244.40.202   123/udp                                  chrony: Ready
      telegraf/2                      active       idle             10.244.40.202   9103/tcp                                 Monitoring grafana/0
    graylog-mongodb/0*                active       idle   10/lxd/0  10.244.40.226   27017/tcp,27019/tcp,27021/tcp,28017/tcp  Unit is ready
      filebeat/14                     active       idle             10.244.40.226                                            Filebeat ready
      nrpe-container/0*               active       idle             10.244.40.226   icmp,5666/tcp                            ready
      telegraf/14                     active       idle             10.244.40.226   9103/tcp                                 Monitoring graylog-mongodb/0
    graylog/0*                        active       idle   10        10.244.40.218   5044/tcp                                 Ready with: filebeat, elasticsearch, mongodb
      canonical-livepatch/12          active       idle             10.244.40.218                                            Running kernel 4.15.0-50.54-generic, patchState: nothing-to-apply
      nrpe-host/12                    active       idle             10.244.40.218   icmp,5666/tcp                            ready
      ntp/13                          active       idle             10.244.40.218   123/udp                                  chrony: Ready
      telegraf/13                     active       idle             10.244.40.218   9103/tcp                                 Monitoring graylog/0
    heat/0                            active       idle   15/lxd/4  10.244.40.246   8000/tcp,8004/tcp                        Unit is ready
      filebeat/34                     active       idle             10.244.40.246                                            Filebeat ready
      hacluster-heat/0*               active       idle             10.244.40.246                                            Unit is ready and clustered
      nrpe-container/14               active       idle             10.244.40.246   icmp,5666/tcp                            ready
      telegraf/34                     active       idle             10.244.40.246   9103/tcp                                 Monitoring heat/0
    heat/1*                           active       idle   16/lxd/5  10.244.40.238   8000/tcp,8004/tcp                        Unit is ready
      filebeat/56                     active       idle             10.244.40.238                                            Filebeat ready.
      hacluster-heat/2                active       idle             10.244.40.238                                            Unit is ready and clustered
      nrpe-container/33               active       idle             10.244.40.238   icmp,5666/tcp                            ready
      telegraf/56                     active       idle             10.244.40.238   9103/tcp                                 Monitoring heat/1
    heat/2                            active       idle   17/lxd/5  10.244.41.0     8000/tcp,8004/tcp                        Unit is ready
      filebeat/43                     active       idle             10.244.41.0                                              Filebeat ready.
      hacluster-heat/1                active       idle             10.244.41.0                                              Unit is ready and clustered
      nrpe-container/22               active       idle             10.244.41.0     icmp,5666/tcp                            ready
      telegraf/43                     active       idle             10.244.41.0     9103/tcp                                 Monitoring heat/2
    keystone/0*                       active       idle   15/lxd/5  10.244.40.243   5000/tcp                                 Unit is ready
      filebeat/33                     active       idle             10.244.40.243                                            Filebeat ready
      hacluster-keystone/0*           active       idle             10.244.40.243                                            Unit is ready and clustered
      keystone-ldap/0*                active       idle             10.244.40.243                                            Unit is ready
      nrpe-container/13               active       idle             10.244.40.243   icmp,5666/tcp                            ready
      telegraf/33                     active       idle             10.244.40.243   9103/tcp                                 Monitoring keystone/0
    keystone/1                        active       idle   16/lxd/6  10.244.40.254   5000/tcp                                 Unit is ready
      filebeat/60                     active       idle             10.244.40.254                                            Filebeat ready
      hacluster-keystone/2            active       idle             10.244.40.254                                            Unit is ready and clustered
      keystone-ldap/2                 active       idle             10.244.40.254                                            Unit is ready
      nrpe-container/37               active       idle             10.244.40.254   icmp,5666/tcp                            ready
      telegraf/60                     active       idle             10.244.40.254   9103/tcp                                 Monitoring keystone/1
    keystone/2                        active       idle   17/lxd/6  10.244.41.3     5000/tcp                                 Unit is ready
      filebeat/48                     active       idle             10.244.41.3                                              Filebeat ready
      hacluster-keystone/1            active       idle             10.244.41.3                                              Unit is ready and clustered
      keystone-ldap/1                 active       idle             10.244.41.3                                              Unit is ready
      nrpe-container/26               active       idle             10.244.41.3     icmp,5666/tcp                            ready
      telegraf/48                     active       idle             10.244.41.3     9103/tcp                                 Monitoring keystone/2
    landscape-haproxy/0*              unknown      idle   2         10.244.40.203   80/tcp,443/tcp
      filebeat/1                      active       idle             10.244.40.203                                            Filebeat ready
      nrpe-host/0*                    active       idle             10.244.40.203   icmp,5666/tcp                            ready
      ntp/1                           active       idle             10.244.40.203   123/udp                                  chrony: Ready
      telegraf/1                      active       idle             10.244.40.203   9103/tcp                                 Monitoring landscape-haproxy/0
    landscape-postgresql/0*           maintenance  idle   3         10.244.40.215   5432/tcp                                 Installing postgresql-.*-debversion,postgresql-plpython-.*
      canonical-livepatch/9           active       idle             10.244.40.215                                            Running kernel 4.15.0-50.54-generic, patchState: nothing-to-apply
      filebeat/10                     active       idle             10.244.40.215                                            Filebeat ready
      nrpe-host/9                     active       idle             10.244.40.215   icmp,5666/tcp                            ready
      ntp/10                          active       idle             10.244.40.215   123/udp                                  chrony: Ready
      telegraf/10                     active       idle             10.244.40.215   9103/tcp                                 Monitoring landscape-postgresql/0
    landscape-postgresql/1            active       idle   8         10.244.40.214   5432/tcp                                 Live secondary (10.8)
      canonical-livepatch/10          active       idle             10.244.40.214                                            Running kernel 4.15.0-50.54-generic, patchState: nothing-to-apply
      filebeat/11                     active       idle             10.244.40.214                                            Filebeat ready
      nrpe-host/10                    active       idle             10.244.40.214   icmp,5666/tcp                            ready
      ntp/11                          active       idle             10.244.40.214   123/udp                                  chrony: Ready
      telegraf/11                     active       idle             10.244.40.214   9103/tcp                                 Monitoring landscape-postgresql/1
    landscape-rabbitmq-server/0*      active       idle   4         10.244.40.211   5672/tcp                                 Unit is ready and clustered
      canonical-livepatch/8           active       idle             10.244.40.211                                            Running kernel 4.15.0-50.54-generic, patchState: nothing-to-apply
      filebeat/9                      active       idle             10.244.40.211                                            Filebeat ready
      nrpe-host/8                     active       idle             10.244.40.211   icmp,5666/tcp                            ready
      ntp/9                           active       idle             10.244.40.211   123/udp                                  chrony: Ready
      telegraf/9                      active       idle             10.244.40.211   9103/tcp                                 Monitoring landscape-rabbitmq-server/0
    landscape-rabbitmq-server/1       active       idle   7         10.244.40.208   5672/tcp                                 Unit is ready and clustered
      canonical-livepatch/11          active       idle             10.244.40.208                                            Running kernel 4.15.0-50.54-generic, patchState: nothing-to-apply
      filebeat/12                     active       idle             10.244.40.208                                            Filebeat ready
      nrpe-host/11                    active       idle             10.244.40.208   icmp,5666/tcp                            ready
      ntp/12                          active       idle             10.244.40.208   123/udp                                  chrony: Ready
      telegraf/12                     active       idle             10.244.40.208   9103/tcp                                 Monitoring landscape-rabbitmq-server/1
    landscape-rabbitmq-server/2       active       idle   12        10.244.40.207   5672/tcp                                 Unit is ready and clustered
      canonical-livepatch/7           active       idle             10.244.40.207                                            Running kernel 4.15.0-50.54-generic, patchState: nothing-to-apply
      filebeat/8                      active       idle             10.244.40.207                                            Filebeat ready
      nrpe-host/7                     active       idle             10.244.40.207   icmp,5666/tcp                            ready
      ntp/8                           active       idle             10.244.40.207   123/udp                                  chrony: Ready
      telegraf/8                      active       idle             10.244.40.207   9103/tcp                                 Monitoring landscape-rabbitmq-server/2
    landscape-server/0*               active       idle   6         10.244.40.210
      canonical-livepatch/4           active       idle             10.244.40.210                                            Running kernel 4.15.0-50.54-generic, patchState: nothing-to-apply
      filebeat/5                      active       idle             10.244.40.210                                            Filebeat ready
      nrpe-host/4                     active       idle             10.244.40.210   icmp,5666/tcp                            ready
      ntp/5                           active       idle             10.244.40.210   123/udp                                  chrony: Ready
      telegraf/5                      active       idle             10.244.40.210   9103/tcp                                 Monitoring landscape-server/0
    landscape-server/1                active       idle   11        10.244.40.212
      canonical-livepatch/5           active       idle             10.244.40.212                                            Running kernel 4.15.0-50.54-generic, patchState: nothing-to-apply
      filebeat/6                      active       idle             10.244.40.212                                            Filebeat ready
      nrpe-host/5                     active       idle             10.244.40.212   icmp,5666/tcp                            ready
      ntp/6                           active       idle             10.244.40.212   123/udp                                  chrony: Ready
      telegraf/6                      active       idle             10.244.40.212   9103/tcp                                 Monitoring landscape-server/1
    landscape-server/2                active       idle   14        10.244.40.204
      canonical-livepatch/6           active       idle             10.244.40.204                                            Running kernel 4.15.0-50.54-generic, patchState: nothing-to-apply
      filebeat/7                      active       idle             10.244.40.204                                            Filebeat ready
      nrpe-host/6                     active       idle             10.244.40.204   icmp,5666/tcp                            ready
      ntp/7                           active       idle             10.244.40.204   123/udp                                  chrony: Ready
      telegraf/7                      active       idle             10.244.40.204   9103/tcp                                 Monitoring landscape-server/2
    memcached/0*                      active       idle   16/lxd/3  10.244.40.250   11211/tcp                                Unit is ready and clustered
    memcached/1                       active       idle   17/lxd/3  10.244.40.255   11211/tcp                                Unit is ready and clustered
    mysql/0*                          active       idle   15/lxd/6  10.244.40.251   3306/tcp                                 Unit is ready
      filebeat/28                     active       idle             10.244.40.251                                            Filebeat ready
      hacluster-mysql/1               active       idle             10.244.40.251                                            Unit is ready and clustered
      nrpe-container/8                active       idle             10.244.40.251   icmp,5666/tcp                            ready
      telegraf/28                     active       idle             10.244.40.251   9103/tcp                                 Monitoring mysql/0
    mysql/1                           active       idle   16/lxd/7  10.244.40.252   3306/tcp                                 Unit is ready
      filebeat/25                     active       idle             10.244.40.252                                            Filebeat ready
      hacluster-mysql/0*              active       idle             10.244.40.252                                            Unit is ready and clustered
      nrpe-container/6                active       idle             10.244.40.252   icmp,5666/tcp                            ready
      telegraf/25                     active       idle             10.244.40.252   9103/tcp                                 Monitoring mysql/1
    mysql/2                           active       idle   17/lxd/7  10.244.41.68    3306/tcp                                 Unit is ready
      filebeat/50                     active       idle             10.244.41.68                                             Filebeat ready
      hacluster-mysql/2               active       idle             10.244.41.68                                             Unit is ready and clustered
      nrpe-container/27               active       idle             10.244.41.68    icmp,5666/tcp                            ready
      telegraf/50                     active       idle             10.244.41.68    9103/tcp                                 Monitoring mysql/2
    nagios/0*                         active       idle   0         10.244.40.201   80/tcp                                   ready
      canonical-livepatch/0*          active       idle             10.244.40.201                                            Running kernel 4.15.0-50.54-generic, patchState: nothing-to-apply
      filebeat/0*                     active       idle             10.244.40.201                                            Filebeat ready
      ntp/0*                          active       idle             10.244.40.201   123/udp                                  chrony: Ready
      telegraf/0*                     active       idle             10.244.40.201   9103/tcp                                 Monitoring nagios/0
      thruk-agent/0*                  unknown      idle             10.244.40.201
    neutron-api/0                     active       idle   18/lxd/4  10.244.41.67    9696/tcp                                 Unit is ready
      filebeat/53                     active       idle             10.244.41.67                                             Filebeat ready
      hacluster-neutron/0*            active       idle             10.244.41.67                                             Unit is ready and clustered
      nrpe-container/30               active       idle             10.244.41.67    icmp,5666/tcp                            ready
      telegraf/53                     active       idle             10.244.41.67    9103/tcp                                 Monitoring neutron-api/0
    neutron-api/1                     active       idle   20/lxd/4  10.244.41.73    9696/tcp                                 Unit is ready
      filebeat/58                     active       idle             10.244.41.73                                             Filebeat ready
      hacluster-neutron/1             active       idle             10.244.41.73                                             Unit is ready and clustered
      nrpe-container/35               active       idle             10.244.41.73    icmp,5666/tcp                            ready
      telegraf/58                     active       idle             10.244.41.73    9103/tcp                                 Monitoring neutron-api/1
    neutron-api/2*                    active       idle   21/lxd/4  10.244.41.6     9696/tcp                                 Unit is ready
      filebeat/64                     active       idle             10.244.41.6                                              Filebeat ready
      hacluster-neutron/2             active       idle             10.244.41.6                                              Unit is ready and clustered
      nrpe-container/41               active       idle             10.244.41.6     icmp,5666/tcp                            ready
      telegraf/64                     active       idle             10.244.41.6     9103/tcp                                 Monitoring neutron-api/2
    neutron-gateway/0                 active       idle   20        10.244.40.224                                            Unit is ready
      canonical-livepatch/21          active       idle             10.244.40.224                                            Running kernel 4.15.0-50.54-generic, patchState: nothing-to-apply
      filebeat/49                     active       idle             10.244.40.224                                            Filebeat ready
      lldpd/8                         active       idle             10.244.40.224                                            LLDP daemon running
      nrpe-host/31                    active       idle             10.244.40.224                                            ready
      ntp/23                          active       idle             10.244.40.224   123/udp                                  chrony: Ready
      telegraf/49                     active       idle             10.244.40.224   9103/tcp                                 Monitoring neutron-gateway/0
    neutron-gateway/1*                active       idle   21        10.244.40.222                                            Unit is ready
      canonical-livepatch/20          active       idle             10.244.40.222                                            Running kernel 4.15.0-50.54-generic, patchState: nothing-to-apply
      filebeat/44                     active       idle             10.244.40.222                                            Filebeat ready
      lldpd/7                         active       idle             10.244.40.222                                            LLDP daemon running
      nrpe-host/28                    active       idle             10.244.40.222   icmp,5666/tcp                            ready
      ntp/22                          active       idle             10.244.40.222   123/udp                                  chrony: Ready
      telegraf/44                     active       idle             10.244.40.222   9103/tcp                                 Monitoring neutron-gateway/1
    nova-cloud-controller/0           active       idle   18/lxd/5  10.244.40.242   8774/tcp,8775/tcp,8778/tcp               Unit is ready
      filebeat/54                     active       idle             10.244.40.242                                            Filebeat ready
      hacluster-nova/1                active       idle             10.244.40.242                                            Unit is ready and clustered
      nrpe-container/31               active       idle             10.244.40.242   icmp,5666/tcp                            ready
      telegraf/54                     active       idle             10.244.40.242   9103/tcp                                 Monitoring nova-cloud-controller/0
    nova-cloud-controller/1           active       idle   20/lxd/5  10.244.41.76    8774/tcp,8775/tcp,8778/tcp               Unit is ready
      filebeat/68                     active       idle             10.244.41.76                                             Filebeat ready
      hacluster-nova/2                active       idle             10.244.41.76                                             Unit is ready and clustered
      nrpe-container/45               active       idle             10.244.41.76    icmp,5666/tcp                            ready
      telegraf/68                     active       idle             10.244.41.76    9103/tcp                                 Monitoring nova-cloud-controller/1
    nova-cloud-controller/2*          active       idle   21/lxd/5  10.244.40.235   8774/tcp,8775/tcp,8778/tcp               Unit is ready
      filebeat/52                     active       idle             10.244.40.235                                            Filebeat ready
      hacluster-nova/0*               active       idle             10.244.40.235                                            Unit is ready and clustered
      nrpe-container/29               active       idle             10.244.40.235   icmp,5666/tcp                            ready
      telegraf/52                     active       idle             10.244.40.235   9103/tcp                                 Monitoring nova-cloud-controller/2
    nova-compute-kvm/0*               active       idle   15        10.244.40.206                                            Unit is ready
      canonical-livepatch/17          active       idle             10.244.40.206                                            Running kernel 4.15.0-50.54-generic, patchState: nothing-to-apply
      ceilometer-agent/4              active       idle             10.244.40.206                                            Unit is ready
      filebeat/23                     active       idle             10.244.40.206                                            Filebeat ready
      lldpd/4                         active       idle             10.244.40.206                                            LLDP daemon running
      neutron-openvswitch/4           active       idle             10.244.40.206                                            Unit is ready
      nrpe-host/22                    active       idle             10.244.40.206                                            ready
      ntp/19                          active       idle             10.244.40.206   123/udp                                  chrony: Ready
      telegraf/23                     active       idle             10.244.40.206   9103/tcp                                 Monitoring nova-compute-kvm/0
    nova-compute-kvm/1                active       idle   16        10.244.40.213                                            Unit is ready
      canonical-livepatch/14          active       idle             10.244.40.213                                            Running kernel 4.15.0-50.54-generic, patchState: nothing-to-apply
      ceilometer-agent/1              active       idle             10.244.40.213                                            Unit is ready
      filebeat/18                     active       idle             10.244.40.213                                            Filebeat ready
      lldpd/1                         active       idle             10.244.40.213                                            LLDP daemon running
      neutron-openvswitch/1           active       idle             10.244.40.213                                            Unit is ready
      nrpe-host/17                    active       idle             10.244.40.213                                            ready
      ntp/16                          active       idle             10.244.40.213   123/udp                                  chrony: Ready
      telegraf/18                     active       idle             10.244.40.213   9103/tcp                                 Monitoring nova-compute-kvm/1
    nova-compute-kvm/2                active       idle   17        10.244.40.220                                            Unit is ready
      canonical-livepatch/18          active       idle             10.244.40.220                                            Running kernel 4.15.0-50.54-generic, patchState: nothing-to-apply
      ceilometer-agent/5              active       idle             10.244.40.220                                            Unit is ready
      filebeat/26                     active       idle             10.244.40.220                                            Filebeat ready
      lldpd/5                         active       idle             10.244.40.220                                            LLDP daemon running
      neutron-openvswitch/5           active       idle             10.244.40.220                                            Unit is ready
      nrpe-host/24                    active       idle             10.244.40.220   icmp,5666/tcp                            ready
      ntp/20                          active       idle             10.244.40.220   123/udp                                  chrony: Ready
      telegraf/26                     active       idle             10.244.40.220   9103/tcp                                 Monitoring nova-compute-kvm/2
    nova-compute-kvm/3                active       idle   18        10.244.40.225                                            Unit is ready
      canonical-livepatch/19          active       idle             10.244.40.225                                            Running kernel 4.15.0-50.54-generic, patchState: nothing-to-apply
      ceilometer-agent/6              active       idle             10.244.40.225                                            Unit is ready
      filebeat/41                     active       idle             10.244.40.225                                            Filebeat ready
      lldpd/6                         active       idle             10.244.40.225                                            LLDP daemon running
      neutron-openvswitch/6           active       idle             10.244.40.225                                            Unit is ready
      nrpe-host/26                    active       idle             10.244.40.225                                            ready
      ntp/21                          active       idle             10.244.40.225   123/udp                                  chrony: Ready
      telegraf/41                     active       idle             10.244.40.225   9103/tcp                                 Monitoring nova-compute-kvm/3
    nova-compute-kvm/4                active       idle   19        10.244.40.221                                            Unit is ready
      canonical-livepatch/15          active       idle             10.244.40.221                                            Running kernel 4.15.0-50.54-generic, patchState: nothing-to-apply
      ceilometer-agent/2              active       idle             10.244.40.221                                            Unit is ready
      filebeat/19                     active       idle             10.244.40.221                                            Filebeat ready
      lldpd/2                         active       idle             10.244.40.221                                            LLDP daemon running
      neutron-openvswitch/2           active       idle             10.244.40.221                                            Unit is ready
      nrpe-host/19                    active       idle             10.244.40.221                                            ready
      ntp/17                          active       idle             10.244.40.221   123/udp                                  chrony: Ready
      telegraf/19                     active       idle             10.244.40.221   9103/tcp                                 Monitoring nova-compute-kvm/4
    nova-compute-lxd/0                active       idle   22        10.244.40.223                                            Unit is ready
      canonical-livepatch/16          active       idle             10.244.40.223                                            Running kernel 4.15.0-50.54-generic, patchState: nothing-to-apply
      ceilometer-agent/3              active       idle             10.244.40.223                                            Unit is ready
      filebeat/20                     active       idle             10.244.40.223                                            Filebeat ready
      lldpd/3                         active       idle             10.244.40.223                                            LLDP daemon running
      neutron-openvswitch/3           active       idle             10.244.40.223                                            Unit is ready
      nrpe-host/21                    active       idle             10.244.40.223                                            ready
      ntp/18                          active       idle             10.244.40.223   123/udp                                  chrony: Ready
      telegraf/20                     active       idle             10.244.40.223   9103/tcp                                 Monitoring nova-compute-lxd/0
    nova-compute-lxd/1*               active       idle   23        10.244.40.219                                            Unit is ready
      canonical-livepatch/13          active       idle             10.244.40.219                                            Running kernel 4.15.0-50.54-generic, patchState: nothing-to-apply
      ceilometer-agent/0*             active       idle             10.244.40.219                                            Unit is ready
      filebeat/16                     active       idle             10.244.40.219                                            Filebeat ready
      lldpd/0*                        active       idle             10.244.40.219                                            LLDP daemon running
      neutron-openvswitch/0*          active       idle             10.244.40.219                                            Unit is ready
      nrpe-host/15                    active       idle             10.244.40.219   icmp,5666/tcp                            ready
      ntp/15                          active       idle             10.244.40.219   123/udp                                  chrony: Ready
      telegraf/16                     active       idle             10.244.40.219   9103/tcp                                 Monitoring nova-compute-lxd/1
    openstack-dashboard/0*            active       idle   18/lxd/6  10.244.40.232   80/tcp,443/tcp                           Unit is ready
      filebeat/30                     active       idle             10.244.40.232                                            Filebeat ready
      hacluster-horizon/0*            active       idle             10.244.40.232                                            Unit is ready and clustered
      nrpe-container/10               active       idle             10.244.40.232   icmp,5666/tcp                            ready
      telegraf/30                     active       idle             10.244.40.232   9103/tcp                                 Monitoring openstack-dashboard/0
    openstack-dashboard/1             active       idle   20/lxd/6  10.244.41.75    80/tcp,443/tcp                           Unit is ready
      filebeat/73                     active       idle             10.244.41.75                                             Filebeat ready
      hacluster-horizon/2             active       idle             10.244.41.75                                             Unit is ready and clustered
      nrpe-container/50               active       idle             10.244.41.75    icmp,5666/tcp                            ready
      telegraf/73                     active       idle             10.244.41.75    9103/tcp                                 Monitoring openstack-dashboard/1
    openstack-dashboard/2             active       idle   21/lxd/6  10.244.41.69    80/tcp,443/tcp                           Unit is ready
      filebeat/72                     active       idle             10.244.41.69                                             Filebeat ready
      hacluster-horizon/1             active       idle             10.244.41.69                                             Unit is ready and clustered
      nrpe-container/49               active       idle             10.244.41.69    icmp,5666/tcp                            ready
      telegraf/72                     active       idle             10.244.41.69    9103/tcp                                 Monitoring openstack-dashboard/2
    openstack-service-checks/0*       active       idle   15/lxd/7  10.244.40.240                                            Unit is ready
      filebeat/31                     active       idle             10.244.40.240                                            Filebeat ready
      nrpe-container/11               active       idle             10.244.40.240   icmp,5666/tcp                            ready
      telegraf/31                     active       idle             10.244.40.240   9103/tcp                                 Monitoring openstack-service-checks/0
    prometheus-ceph-exporter/0*       active       idle   16/lxd/8  10.244.40.245   9128/tcp                                 Running
      filebeat/38                     active       idle             10.244.40.245                                            Filebeat ready
      nrpe-container/18               active       idle             10.244.40.245   icmp,5666/tcp                            ready
      telegraf/38                     active       idle             10.244.40.245   9103/tcp                                 Monitoring prometheus-ceph-exporter/0
    prometheus-openstack-exporter/0*  active       idle   17/lxd/8  10.244.41.1                                              Ready
      filebeat/39                     active       idle             10.244.41.1                                              Filebeat ready
      nrpe-container/19               active       idle             10.244.41.1     icmp,5666/tcp                            ready
      telegraf/39                     active       idle             10.244.41.1     9103/tcp                                 Monitoring prometheus-openstack-exporter/0
    prometheus/0*                     active       idle   9         10.244.40.216   9090/tcp,12321/tcp                       Ready
      filebeat/13                     active       idle             10.244.40.216                                            Filebeat ready
      nrpe-host/13                    active       idle             10.244.40.216   icmp,5666/tcp                            ready
      ntp/14                          active       idle             10.244.40.216   123/udp                                  chrony: Ready
      telegraf-prometheus/0*          active       idle             10.244.40.216   9103/tcp                                 Monitoring prometheus/0
    rabbitmq-server/0                 active       idle   18/lxd/7  10.244.41.65    5672/tcp                                 Unit is ready and clustered
      filebeat/62                     active       idle             10.244.41.65                                             Filebeat ready
      nrpe-container/39               active       idle             10.244.41.65    icmp,5666/tcp                            ready
      telegraf/62                     active       idle             10.244.41.65    9103/tcp                                 Monitoring rabbitmq-server/0
    rabbitmq-server/1*                active       idle   20/lxd/7  10.244.40.247   5672/tcp                                 Unit is ready and clustered
      filebeat/32                     active       idle             10.244.40.247                                            Filebeat ready
      nrpe-container/12               active       idle             10.244.40.247   icmp,5666/tcp                            ready
      telegraf/32                     active       idle             10.244.40.247   9103/tcp                                 Monitoring rabbitmq-server/1
    rabbitmq-server/2                 active       idle   21/lxd/7  10.244.41.4     5672/tcp                                 Unit is ready and clustered
      filebeat/66                     active       idle             10.244.41.4                                              Filebeat ready
      nrpe-container/43               active       idle             10.244.41.4     icmp,5666/tcp                            ready
      telegraf/67                     active       idle             10.244.41.4     9103/tcp                                 Monitoring rabbitmq-server/2

    Machine   State    DNS            Inst id               Series  AZ       Message
    0         started  10.244.40.201  nagios-1              bionic  default  Deployed
    1         started  10.244.40.202  grafana-1             bionic  default  Deployed
    2         started  10.244.40.203  landscapeha-1         bionic  default  Deployed
    3         started  10.244.40.215  landscapesql-1        bionic  default  Deployed
    4         started  10.244.40.211  landscapeamqp-1       bionic  default  Deployed
    5         started  10.244.40.217  elastic-3             bionic  zone3    Deployed
    6         started  10.244.40.210  landscape-2           bionic  zone2    Deployed
    7         started  10.244.40.208  landscapeamqp-3       bionic  zone3    Deployed
    8         started  10.244.40.214  landscapesql-2        bionic  zone2    Deployed
    9         started  10.244.40.216  prometheus-3          bionic  zone3    Deployed
    10        started  10.244.40.218  graylog-3             bionic  zone3    Deployed
    10/lxd/0  started  10.244.40.226  juju-5aed61-10-lxd-0  bionic  zone3    Container started
    11        started  10.244.40.212  landscape-3           bionic  zone3    Deployed
    12        started  10.244.40.207  landscapeamqp-2       bionic  zone2    Deployed
    13        started  10.244.40.209  elastic-2             bionic  zone2    Deployed
    14        started  10.244.40.204  landscape-1           bionic  default  Deployed
    15        started  10.244.40.206  suicune               bionic  zone2    Deployed
    15/lxd/0  started  10.244.40.227  juju-5aed61-15-lxd-0  bionic  zone2    Container started
    15/lxd/1  started  10.244.40.228  juju-5aed61-15-lxd-1  bionic  zone2    Container started
    15/lxd/2  started  10.244.40.249  juju-5aed61-15-lxd-2  bionic  zone2    Container started
    15/lxd/3  started  10.244.40.237  juju-5aed61-15-lxd-3  bionic  zone2    Container started
    15/lxd/4  started  10.244.40.246  juju-5aed61-15-lxd-4  bionic  zone2    Container started
    15/lxd/5  started  10.244.40.243  juju-5aed61-15-lxd-5  bionic  zone2    Container started
    15/lxd/6  started  10.244.40.251  juju-5aed61-15-lxd-6  bionic  zone2    Container started
    15/lxd/7  started  10.244.40.240  juju-5aed61-15-lxd-7  bionic  zone2    Container started
    16        started  10.244.40.213  geodude               bionic  default  Deployed
    16/lxd/0  started  10.244.40.253  juju-5aed61-16-lxd-0  bionic  default  Container started
    16/lxd/1  started  10.244.40.241  juju-5aed61-16-lxd-1  bionic  default  Container started
    16/lxd/2  started  10.244.40.248  juju-5aed61-16-lxd-2  bionic  default  Container started
    16/lxd/3  started  10.244.40.250  juju-5aed61-16-lxd-3  bionic  default  Container started
    16/lxd/4  started  10.244.41.5    juju-5aed61-16-lxd-4  bionic  default  Container started
    16/lxd/5  started  10.244.40.238  juju-5aed61-16-lxd-5  bionic  default  Container started
    16/lxd/6  started  10.244.40.254  juju-5aed61-16-lxd-6  bionic  default  Container started
    16/lxd/7  started  10.244.40.252  juju-5aed61-16-lxd-7  bionic  default  Container started
    16/lxd/8  started  10.244.40.245  juju-5aed61-16-lxd-8  bionic  default  Container started
    17        started  10.244.40.220  armaldo               bionic  default  Deployed
    17/lxd/0  started  10.244.41.78   juju-5aed61-17-lxd-0  bionic  default  Container started
    17/lxd/1  started  10.244.40.233  juju-5aed61-17-lxd-1  bionic  default  Container started
    17/lxd/2  started  10.244.41.2    juju-5aed61-17-lxd-2  bionic  default  Container started
    17/lxd/3  started  10.244.40.255  juju-5aed61-17-lxd-3  bionic  default  Container started
    17/lxd/4  started  10.244.40.234  juju-5aed61-17-lxd-4  bionic  default  Container started
    17/lxd/5  started  10.244.41.0    juju-5aed61-17-lxd-5  bionic  default  Container started
    17/lxd/6  started  10.244.41.3    juju-5aed61-17-lxd-6  bionic  default  Container started
    17/lxd/7  started  10.244.41.68   juju-5aed61-17-lxd-7  bionic  default  Container started
    17/lxd/8  started  10.244.41.1    juju-5aed61-17-lxd-8  bionic  default  Container started
    18        started  10.244.40.225  elgyem                bionic  zone3    Deployed
    18/lxd/0  started  10.244.40.236  juju-5aed61-18-lxd-0  bionic  zone3    Container started
    18/lxd/1  started  10.244.40.239  juju-5aed61-18-lxd-1  bionic  zone3    Container started
    18/lxd/2  started  10.244.41.70   juju-5aed61-18-lxd-2  bionic  zone3    Container started
    18/lxd/3  started  10.244.40.231  juju-5aed61-18-lxd-3  bionic  zone3    Container started
    18/lxd/4  started  10.244.41.67   juju-5aed61-18-lxd-4  bionic  zone3    Container started
    18/lxd/5  started  10.244.40.242  juju-5aed61-18-lxd-5  bionic  zone3    Container started
    18/lxd/6  started  10.244.40.232  juju-5aed61-18-lxd-6  bionic  zone3    Container started
    18/lxd/7  started  10.244.41.65   juju-5aed61-18-lxd-7  bionic  zone3    Container started
    19        started  10.244.40.221  spearow               bionic  zone2    Deployed
    20        started  10.244.40.224  quilava               bionic  default  Deployed
    20/lxd/0  started  10.244.41.74   juju-5aed61-20-lxd-0  bionic  default  Container started
    20/lxd/1  started  10.244.41.77   juju-5aed61-20-lxd-1  bionic  default  Container started
    20/lxd/2  started  10.244.41.72   juju-5aed61-20-lxd-2  bionic  default  Container started
    20/lxd/3  started  10.244.40.244  juju-5aed61-20-lxd-3  bionic  default  Container started
    20/lxd/4  started  10.244.41.73   juju-5aed61-20-lxd-4  bionic  default  Container started
    20/lxd/5  started  10.244.41.76   juju-5aed61-20-lxd-5  bionic  default  Container started
    20/lxd/6  started  10.244.41.75   juju-5aed61-20-lxd-6  bionic  default  Container started
    20/lxd/7  started  10.244.40.247  juju-5aed61-20-lxd-7  bionic  default  Container started
    21        started  10.244.40.222  rufflet               bionic  zone3    Deployed
    21/lxd/0  started  10.244.41.66   juju-5aed61-21-lxd-0  bionic  zone3    Container started
    21/lxd/1  started  10.244.40.229  juju-5aed61-21-lxd-1  bionic  zone3    Container started
    21/lxd/2  started  10.244.41.71   juju-5aed61-21-lxd-2  bionic  zone3    Container started
    21/lxd/3  started  10.244.40.230  juju-5aed61-21-lxd-3  bionic  zone3    Container started
    21/lxd/4  started  10.244.41.6    juju-5aed61-21-lxd-4  bionic  zone3    Container started
    21/lxd/5  started  10.244.40.235  juju-5aed61-21-lxd-5  bionic  zone3    Container started
    21/lxd/6  started  10.244.41.69   juju-5aed61-21-lxd-6  bionic  zone3    Container started
    21/lxd/7  started  10.244.41.4    juju-5aed61-21-lxd-7  bionic  zone3    Container started
    22        started  10.244.40.223  ralts                 bionic  zone2    Deployed
    23        started  10.244.40.219  beartic               bionic  zone3    Deployed
