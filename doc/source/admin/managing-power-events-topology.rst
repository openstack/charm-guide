:orphan:

.. _cloud_topology_example:

Cloud topology example
======================

.. note::

    The information on this page is associated with the topic of :ref:`Managing
    Power Events <managing_power_events>`. See that page for background
    information.

This page contains the analysis of cloud machines. The ideal is to do this for
every machine in a cloud in order to determine the *cloud topology*. Six
machines are features here. They represent a good cross-section of an *Ubuntu
OpenStack* cloud. See :ref:`Reference cloud <reference_cloud>` for the cloud
upon which this exercise is based.

Generally speaking, the cloud nodes are hyperconverged and this is the case for
three of the chosen machines, numbered **17**, **18**, and **20**. Yet this
analysis also looks at a trio of nodes dedicated to the `Landscape project`_:
machines **3**, **11**, and **12**, each of which are not hyperconverged.

.. note::

   A Juju application can be given a custom name (i.e. named application) at
   deploy time. When multiple units are involved this is referred to as an
   application group.

**machine 17**

This is what's on machine 17:

.. code::

    Unit                              Workload     Agent  Machine
    nova-compute-kvm/2                active       idle   17
      canonical-livepatch/18          active       idle
      ceilometer-agent/5              active       idle
      filebeat/26                     active       idle
      lldpd/5                         active       idle
      neutron-openvswitch/5           active       idle
      nrpe-host/24                    active       idle
      ntp/20                          active       idle
      telegraf/26                     active       idle
    ceph-osd/2                        active       idle   17
      bcache-tuning/4                 active       idle
      nrpe-host/23                    active       idle
    ceph-mon/2                        active       idle   17/lxd/0
      filebeat/71                     active       idle
      nrpe-container/48               active       idle
      telegraf/71                     active       idle
    ceph-radosgw/2                    active       idle   17/lxd/1
      filebeat/21                     active       idle
      hacluster-radosgw/1             active       idle
      nrpe-container/3                active       idle
      telegraf/21                     active       idle
    cinder/2                          active       idle   17/lxd/2
      cinder-ceph/1                   active       idle
      filebeat/42                     active       idle
      hacluster-cinder/1              active       idle
      nrpe-container/21               active       idle
      telegraf/42                     active       idle
    designate-bind/1                  active       idle   17/lxd/3
      filebeat/40                     active       idle
      nrpe-container/20               active       idle
      telegraf/40                     active       idle
    glance/2*                         active       idle   17/lxd/4
      filebeat/37                     active       idle
      hacluster-glance/1              active       idle
      nrpe-container/17               active       idle
      telegraf/37                     active       idle
    heat/2                            active       idle   17/lxd/5
      filebeat/43                     active       idle
      hacluster-heat/1                active       idle
      nrpe-container/22               active       idle
      telegraf/43                     active       idle
    keystone/2                        active       idle   17/lxd/6
      filebeat/48                     active       idle
      hacluster-keystone/1            active       idle
      keystone-ldap/1                 active       idle
      nrpe-container/26               active       idle
      telegraf/48                     active       idle
    mysql/2                           active       idle   17/lxd/7
      filebeat/50                     active       idle
      hacluster-mysql/2               active       idle
      nrpe-container/27               active       idle
      telegraf/50                     active       idle
    prometheus-openstack-exporter/0*  active       idle   17/lxd/8
      filebeat/39                     active       idle
      nrpe-container/19               active       idle
      telegraf/39                     active       idle

.. attention::

    In this example, ``mysql`` and ``nova-compute-kvm`` are `named
    applications`. The rest of this section will use their real names of
    ``percona-cluster`` and ``nova-compute``, respectively.

The main applications (principle charms) for this machine are listed below
along with their HA status and machine type:

- ``nova-compute`` (metal)
- ``ceph-osd`` (natively HA; metal)
- ``ceph-mon`` (natively HA; lxd)
- ``ceph-radosgw`` (natively HA; lxd)
- ``cinder`` (HA; lxd)
- ``designate-bind`` (HA; lxd)
- ``glance`` (HA; lxd)
- ``heat`` (HA; lxd)
- ``keystone`` (HA; lxd)
- ``percona-cluster`` (HA; lxd)
- ``prometheus-openstack-exporter`` (lxd)

**machine 18**

This is what's on machine 18:

.. code::

    Unit                              Workload     Agent  Machine
    nova-compute-kvm/3                active       idle   18
      canonical-livepatch/19          active       idle
      ceilometer-agent/6              active       idle
      filebeat/41                     active       idle
      lldpd/6                         active       idle
      neutron-openvswitch/6           active       idle
      nrpe-host/26                    active       idle
      ntp/21                          active       idle
      telegraf/41                     active       idle
    ceph-osd/3                        active       idle   18
      bcache-tuning/5                 active       idle
      nrpe-host/25                    active       idle
    aodh/0*                           active       idle   18/lxd/0
      filebeat/46                     active       idle
      hacluster-aodh/0*               active       idle
      nrpe-container/24               active       idle
      telegraf/46                     active       idle
    ceilometer/0                      blocked      idle   18/lxd/1
      filebeat/51                     active       idle
      hacluster-ceilometer/1          active       idle
      nrpe-container/28               active       idle
      telegraf/51                     active       idle
    designate/0*                      active       idle   18/lxd/2
      filebeat/57                     active       idle
      hacluster-designate/0*          active       idle
      nrpe-container/34               active       idle
      telegraf/57                     active       idle
    gnocchi/0                         active       idle   18/lxd/3
      filebeat/24                     active       idle
      hacluster-gnocchi/0*            active       idle
      nrpe-container/5                active       idle
      telegraf/24                     active       idle
    neutron-api/0                     active       idle   18/lxd/4
      filebeat/53                     active       idle
      hacluster-neutron/0*            active       idle
      nrpe-container/30               active       idle
      telegraf/53                     active       idle
    nova-cloud-controller/0           active       idle   18/lxd/5
      filebeat/54                     active       idle
      hacluster-nova/1                active       idle
      nrpe-container/31               active       idle
      telegraf/54                     active       idle
    openstack-dashboard/0*            active       idle   18/lxd/6
      filebeat/30                     active       idle
      hacluster-horizon/0*            active       idle
      nrpe-container/10               active       idle
      telegraf/30                     active       idle
    rabbitmq-server/0                 active       idle   18/lxd/7
      filebeat/62                     active       idle
      nrpe-container/39               active       idle
      telegraf/62                     active       idle

.. attention::

    In this example, ``nova-compute-kvm`` is a `named application` The rest of
    this section will use its real name of ``nova-compute``.

The main applications (principle charms) for this machine are listed below
along with their HA status and machine type:

- ``nova-compute`` (metal)
- ``ceph-osd`` (natively HA; metal)
- ``aodh`` (HA; lxd)
- ``ceilometer`` (HA; lxd)
- ``designate`` (HA; lxd)
- ``gnocchi`` (HA; lxd)
- ``neutron-api`` (HA; lxd)
- ``nova-cloud-controller`` (HA; lxd)
- ``openstack-dashboard`` (HA; lxd)
- ``rabbitmq-server`` (natively HA; lxd)

**machine 20**

This is what's on machine 20:

.. code::

    Unit                              Workload     Agent  Machine
    neutron-gateway/0                 active       idle   20
      canonical-livepatch/21          active       idle
      filebeat/49                     active       idle
      lldpd/8                         active       idle
      nrpe-host/31                    active       idle
      ntp/23                          active       idle
      telegraf/49                     active       idle
    ceph-osd/5                        active       idle   20
      bcache-tuning/6                 active       idle
      nrpe-host/27                    active       idle
    aodh/1                            active       idle   20/lxd/0
      filebeat/61                     active       idle
      hacluster-aodh/1                active       idle
      nrpe-container/38               active       idle
      telegraf/61                     active       idle
    ceilometer/1                      blocked      idle   20/lxd/1
      filebeat/70                     active       idle
      hacluster-ceilometer/2          active       idle
      nrpe-container/47               active       idle
      telegraf/70                     active       idle
    designate/1                       active       idle   20/lxd/2
      filebeat/63                     active       idle
      hacluster-designate/1           active       idle
      nrpe-container/40               active       idle
      telegraf/63                     active       idle
    gnocchi/1                         active       idle   20/lxd/3
      filebeat/55                     active       idle
      hacluster-gnocchi/2             active       idle
      nrpe-container/32               active       idle
      telegraf/55                     active       idle
    neutron-api/1                     active       idle   20/lxd/4
      filebeat/58                     active       idle
      hacluster-neutron/1             active       idle
      nrpe-container/35               active       idle
      telegraf/58                     active       idle
    nova-cloud-controller/1           active       idle   20/lxd/5
      filebeat/68                     active       idle
      hacluster-nova/2                active       idle
      nrpe-container/45               active       idle
      telegraf/68                     active       idle
    openstack-dashboard/1             active       idle   20/lxd/6
      filebeat/73                     active       idle
      hacluster-horizon/2             active       idle
      nrpe-container/50               active       idle
      telegraf/73                     active       idle
    rabbitmq-server/1*                active       idle   20/lxd/7
      filebeat/32                     active       idle
      nrpe-container/12               active       idle
      telegraf/32                     active       idle

The main applications (principle charms) for this machine are listed below
along with their HA status and machine type:

- ``neutron-gateway`` (natively HA; metal)
- ``ceph-osd`` (natively HA; metal)
- ``aodh`` (HA; lxd)
- ``ceilometer`` (HA; lxd)
- ``designate`` (HA; lxd)
- ``gnocchi`` (HA; lxd)
- ``neutron-api`` (HA; lxd)
- ``nova-cloud-controller`` (HA; lxd)
- ``openstack-dashboard`` (HA; lxd)
- ``rabbitmq-server`` (natively HA; lxd)

**machine 3**

This is what's on machine 3:

.. code::

    Unit                              Workload     Agent  Machine
    landscape-postgresql/0*           maintenance  idle   3
      canonical-livepatch/9           active       idle
      filebeat/10                     active       idle
      nrpe-host/9                     active       idle
      ntp/10                          active       idle
      telegraf/10                     active       idle

.. attention::

    In this example, ``landscape-postgresql`` is a `named application` The rest
    of this section will use its real name of ``postgresql``.

The main application (principle charm) for this machine is listed below along
along with their HA status and machine type:

- ``postgresql`` (natively HA; metal)

**machine 11**

This is what's on machine 11:

.. code::

    Unit                              Workload     Agent  Machine
    landscape-server/1                active       idle   11
      canonical-livepatch/5           active       idle
      filebeat/6                      active       idle
      nrpe-host/5                     active       idle
      ntp/6                           active       idle
      telegraf/6                      active       idle

The main application (principle charm) for this machine is listed below along
along with their HA status and machine type:

- ``landscape-server`` (natively HA; metal)

**machine 12**

This is what's on machine 12:

.. code::

    Unit                              Workload     Agent  Machine
    landscape-rabbitmq-server/2       active       idle   12
      canonical-livepatch/7           active       idle
      filebeat/8                      active       idle
      nrpe-host/7                     active       idle
      ntp/8                           active       idle
      telegraf/8                      active       idle

.. attention::

    In this example, ``landscape-rabbitmq-server`` is a `named application`.
    The rest of this section will use its real name of ``rabbitmq-server``.

The main application (principle charm) for this machine is listed below along
along with their HA status and machine type:

- ``rabbitmq-server`` (natively HA; metal)

.. LINKS
.. _Landscape project: https://landscape.canonical.com
