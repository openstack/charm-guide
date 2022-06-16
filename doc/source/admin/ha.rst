================================
Infrastructure high availability
================================

Overview
--------

Prior to deploying your OpenStack cloud you will need to consider the critical
aspect of high availability (HA) as this will dictate the cloud's topology.
This guide discusses the basics of high availability, how Charmed OpenStack
delivers HA, and any ramifications for the operator once the cloud is deployed.

In addition, any `known issues`_ affecting highly available OpenStack
applications are documented. Although they are presented last, it is
recommended to review them prior to attempting to apply any of the information
shown here.

.. important::

   This document assumes that the OpenStack cloud is backed by `MAAS`_.

Cloud topology guidelines
-------------------------

Deploying applications under HA will involve multiple application units being
distributed amongst the available cloud nodes, thereby dramatically influencing
the cloud topology. The output of the :command:`juju status` command is a
natural way for Juju operators to view their cloud's topology.

MAAS zones and failure domains
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

A fundamental part of achieving an optimal cloud topology is to have the
underlying cloud nodes organised in terms of hardware failure domains. This
means assigning MAAS nodes to `MAAS zones`_ so that each zone comprises a
failure domain.

.. note::

   A failure domain is often defined as a collection of physical devices that
   use common power, cooling, and networking hardware. This often corresponds
   to a data centre machine rack.

MAAS guarantees at least one zone to exist (the 'default' zone) but the minimum
for HA is three. A MAAS zone is exposed in the :command:`juju status` command
output (see the AZ column) and is to be interpreted as an "HA zone" which
should be associated with a complete set of HA cloud services.

Unit distribution
~~~~~~~~~~~~~~~~~

Juju distributes an application's units across zones by default. For example,
if a three-unit application is deployed within a three-zone MAAS environment,
each unit will be assigned to a different zone, and thereby to a different
failure domain.

This behaviour can be overridden by assigning units to specific machines. This
is done through the use of a placement directive (the ``--to`` option available
to the :command:`juju deploy` or :command:`juju add-unit` commands) or the
'zones' machine constraint (e.g. ``--constraints zones=zone2``). This can be
done at deploy or scale out time and is particularly useful for hyperconverged
cloud nodes. See `How to deploy to a specific machine`_ and `Bundle reference`_
in the Juju documentation.

.. note::

   When a LXD container is the targeted machine, its zone is inherited from its
   host machine.

Whether using the default behaviour or the manual placement method, the
resulting units will show in :command:`juju status` command output as being in
the corresponding MAAS node's zone (AZ column).

.. note::

   Charmed OpenStack cloud deployments generally use hyperconverged nodes and
   make extensive use of the placement directive in combination with bundles
   (and possibly bundle overlays).

Availability Zones
~~~~~~~~~~~~~~~~~~

The term "availability zone" can take on different meanings depending on the
context and/or the software implementation. Below we'll see mention of the
following terms:

* Juju AZ (a general HA zone, or failure domain)
* Neutron AZ
* Nova AZ
* Ceph AZ

.. note::

   The term "zone" used in the `swift-proxy`_ and `swift-storage`_ charms
   refers specifically to internal functionality of Swift. Similarly, terms
   "zone" and "zonegroup" used in the `ceph-radosgw`_ charm refer specifically
   to the internal functionality of RADOS Gateway.

JUJU_AVAILABILITY_ZONE
^^^^^^^^^^^^^^^^^^^^^^

The internal Juju variable JUJU_AVAILABILITY_ZONE provides further availability
zone awareness to a unit, however both the backing cloud and the charm need to
support it. For the purposes of this document, this variable associates a MAAS
zone with a unit at deploy time.

This feature is enabled via configuration option ``customize-failure-domain``
for the charms that support it.

Neutron AZ
^^^^^^^^^^

This section pertains to the `neutron-gateway`_ charm.

Configuration option ``default-availability-zone`` sets a single default
`Neutron availability zone`_ to use for Neutron agents (DHCP and L3) when a
network or router is defined with multiple sets of these agents. The default
value is 'nova'.

When option ``customize-failure-domain`` is set to 'true' then all MAAS-defined
zones will become available as Neutron availability zones. In the absence of a
client-specified AZ during router/network creation, the Neutron agents will be
distributed amongst the zones. When 'true', and MAAS is the backing cloud, this
option overrides option ``default-availability-zone``.

These options also affect the `neutron-openvswitch`_ subordinate charm as AZ
information is passed over the relation it forms with the nova-compute charm.
This is useful for Neutron agent scheduling.

.. note::

   The OVN charms do not currently support the configuration of Neutron AZs.

Nova AZ
^^^^^^^

This section pertains to the `nova-compute`_ charm.

Configuration option ``default-availability-zone`` sets a single default `Nova
availability zone`_. It is used when an OpenStack instance is created without a
Nova AZ being specified. The default value is 'nova'. Note that such a Nova AZ
must be created manually (i.e. command :command:`openstack aggregate create`).

When option ``customize-failure-domain`` is set to 'true' then all MAAS-defined
zones will become available as Nova availability zones. In the absence of a
client-specified AZ during instance creation, one of these zones will be
scheduled. When 'true', and MAAS is the backing cloud, this option overrides
option ``default-availability-zone``.

Ceph AZ
^^^^^^^

This section pertains to the `ceph-osd`_ charm.

Configuration option ``availability_zone`` sets a single availability zone for
OSD location. The use of this option implies a very manual approach to
constructing a Ceph CRUSH map and is therefore not recommended.

When option ``customize-failure-domain`` is set to 'false' (the default) a Ceph
CRUSH map will be generated that will replicate data across hosts (implemented
as `Ceph bucket type`_ 'host').

When option ``customize-failure-domain`` is set to 'true' then all MAAS-defined
zones will be used to generate a Ceph CRUSH map that will replicate data across
Ceph availability zones (implemented as bucket type 'rack'). This option is
also supported by the `ceph-mon`_ charm and both charms must give it the same
value. When 'true', this option overrides option ``availability_zone``.

Containerisation
~~~~~~~~~~~~~~~~

Generally speaking, every major OpenStack application can be placed into a LXD
container with the following exceptions:

* ceph-osd
* neutron-gateway
* nova-compute
* swift-storage

Containerisation is effective for scaling out and it renders complex cloud
upgrades manageable. Mapping applications to machines is exceptionally
convenient.

Applications that have been configured to utilise another storage solution as
their backend, such as Ceph, are often containerised. Common applications in
this category include:

* cinder
* glance

HA applications
---------------

This section provides an overview of HA applications. Deployment details are
provided in the section following.

An HA-enabled application is resistant to disruptions affecting its other
cluster members. That is, such a disruption would have no impact on both client
requests to the application and the application itself.

.. note::

   Highly available applications may require attention if subjected to a power
   event (see `Managing power events`_ in the Admin Guide).

Cloud applications are typically made highly available through the use of
techniques applied externally to the application itself (e.g. using a
subordinate charm). Some applications, though, achieve HA via the application's
built-in capabilities, and can be called *natively HA*.

.. important::

   The nova-compute application cannot be made highly available. However, see
   :doc:`Instance high availability <instance-ha>` for an implementation of
   cloud instance HA.

Native HA
~~~~~~~~~

OpenStack service and applications that support native HA are listed here:

+----------+--------------------------+--------------------------------------------------------------------------------------------------------+
| Service  | Application/Charm        | Comments                                                                                               |
+==========+==========================+========================================================================================================+
| Ceph     | ceph-mon, ceph-osd       |                                                                                                        |
+----------+--------------------------+--------------------------------------------------------------------------------------------------------+
| MySQL    | percona-cluster          | MySQL 5.x; external HA technique required for client access; available prior to Ubuntu 20.04 LTS       |
+----------+--------------------------+--------------------------------------------------------------------------------------------------------+
| MySQL    | mysql-innodb-cluster     | MySQL 8.x; used starting with Ubuntu 20.04 LTS                                                         |
+----------+--------------------------+--------------------------------------------------------------------------------------------------------+
| OVN      | ovn-central, ovn-chassis | OVN is HA by design; available starting with Ubuntu 18.04 LTS and Ubuntu 20.04 LTS on OpenStack Ussuri |
+----------+--------------------------+--------------------------------------------------------------------------------------------------------+
| RabbitMQ | rabbitmq-server          |                                                                                                        |
+----------+--------------------------+--------------------------------------------------------------------------------------------------------+
| Swift    | swift-storage            |                                                                                                        |
+----------+--------------------------+--------------------------------------------------------------------------------------------------------+

Non-native HA
~~~~~~~~~~~~~

There are two mutually exclusive strategies when implementing high availability
for applications that do not support it natively:

* virtual IP(s)
* DNS

In both cases, the hacluster subordinate charm is required. It provides the
Corosync/Pacemaker backend HA functionality.

.. note::

   The virtual IP (VIP) method is intended for use in MAAS managed
   environments.

virtual IP(s)
^^^^^^^^^^^^^

To use virtual IP(s) the clustered nodes and the VIP must be on the same
subnet. That is, the VIP must be a valid IP on the subnet for one of the node's
interfaces and each node has an interface in that subnet.

The VIP therefore becomes a highly-available API endpoint and is defined via
the principle charm configuration option ``vip``. Its value can take on
space-separated IP addresses if multiple networks are in use.

Generic deployment commands for a three-unit cluster are provided below.

.. code-block:: none

   juju deploy -n 3 --config vip=<ip-address> <charm-name>
   juju deploy --config cluster_count=3 hacluster <charm-name>-hacluster
   juju add-relation <charm-name>-hacluster:ha <charm-name>:ha

The hacluster application name was chosen as '<charm-name>-hacluster'. This is
the recommended notation.

.. note::

   The default value of option ``cluster_count`` is '3', but it is best
   practice to provide a value explicitly.

DNS
^^^

DNS high availability does not require the clustered nodes to be on the same
subnet, and as such is suitable for use in routed network design where L2
broadcast domains terminate at the "top-of-rack" switch.

It does require:

* an environment with MAAS 2.0 and Juju 2.0 (as minimum versions)
* clustered nodes with static or "reserved" IP addresses registered in MAAS
* DNS hostnames pre-registered in MAAS (if MAAS < 2.3)

At a minimum, the configuration option ``dns-ha`` must be set to 'true' and at
least one of ``os-admin-hostname``, ``os-internal-hostname``, or
``os-public-hostname`` must be set.

An error will occur if:

* neither ``vip`` nor ``dns-ha`` is set and the charm has a relation added to
  hacluster
* both ``vip`` and ``dns-ha`` are set
* ``dns-ha`` is set and none of ``os-admin-hostname``,
  ``os-internal-hostname``, or ``os-public-hostname`` are set

.. caution::

   DNS HA has been reported to not work on the focal series. See `LP #1882508`_
   for more information.

Deployment of HA applications
-----------------------------

This section provides instructions for deploying common native HA and
non-native HA applications. Its sub-sections are not meant to be followed as a
guide on how to deploy a cloud. They are a collection of examples only.

Any relations needed in order for other applications to work with the deployed
HA applications are not considered unless they aid in demonstrating an
exceptional aspect of the HA application's deployment.

Keystone - hacluster
~~~~~~~~~~~~~~~~~~~~

Keystone provides a classic use case of a non-native HA application.

These commands will deploy a three-node Keystone HA cluster, with a VIP of
10.246.114.11. Each node will reside in a container on existing machines 0, 1,
and 2:

.. code-block:: none

   juju deploy -n 3 --to lxd:0,lxd:1,lxd:2 --config vip=10.246.114.11 keystone
   juju deploy --config cluster_count=3 hacluster keystone-hacluster
   juju add-relation keystone-hacluster:ha keystone:ha

This is sample (partial) output from the :command:`juju status` command
resulting from such a deployment:

.. code-block:: console

   Unit                     Workload  Agent  Machine  Public address  Ports     Message
   keystone/0*              active    idle   0/lxd/0  10.246.114.59   5000/tcp  Unit is ready
     keystone-hacluster/0   active    idle            10.246.114.59             Unit is ready and clustered
   keystone/1               active    idle   1/lxd/0  10.246.114.60   5000/tcp  Unit is ready
     keystone-hacluster/2*  active    idle            10.246.114.60             Unit is ready and clustered
   keystone/2               active    idle   2/lxd/0  10.246.114.61   5000/tcp  Unit is ready
     keystone-hacluster/1   active    idle            10.246.114.61             Unit is ready and clustered

The VIP is not exposed in this output.

.. note::

   The unit numbers of the hacluster subordinate and its parent do not
   necessarily coincide. In the above example, only for keystone/0 does this
   occur. That is, keystone-hacluster/0 is the subordinate unit of keystone/0.

To add a relation between an hacluster-enabled application and another
OpenStack application proceed as if hacluster was not involved. For Cinder:

.. code-block:: none

   juju add-relation keystone:identity-service cinder:identity-service

Nova Cloud Controller - hacluster
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Nova Cloud Controller provides a use case of a non-native HA application that
requires an extra application in addition to hacluster: memcached.

.. note::

   Memcached is needed in order for the nova-cloud-controller units to share
   authentication tokens, but please see `LP #1958674`_ for more clarity.

These commands will deploy a three-node Nova Cloud Controller HA cluster:

.. code-block:: none

   juju deploy -n 3 --to lxd:0,lxd:1,lxd:2 --config vip=10.246.114.12 nova-cloud-controller
   juju deploy --config cluster_count=3 hacluster nova-cloud-controller-hacluster
   juju add-relation nova-cloud-controller-hacluster:ha nova-cloud-controller:ha

The memcached application is then deployed (here containerised on machine 0)
and related to nova-cloud-controller:

.. code-block:: none

   juju deploy --to lxd:0 memcached
   juju add-relation memcached:cache nova-cloud-controller:memcache

This is sample (partial) output from the :command:`juju status` command
resulting from such a deployment:

.. code-block:: console

   Unit                                  Workload  Agent  Machine  Public address  Ports              Message
   memcached/0*                          active    idle   0/lxd/5  10.246.114.57   11211/tcp          Unit is ready
   nova-cloud-controller/0*              active    idle   0/lxd/3  10.246.114.32   8774/tcp,8775/tcp  Unit is ready
     nova-cloud-controller-hacluster/0*  active    idle            10.246.114.32                      Unit is ready and clustered
   nova-cloud-controller/1               active    idle   1/lxd/5  10.246.114.56   8774/tcp,8775/tcp  Unit is ready
     nova-cloud-controller-hacluster/2   active    idle            10.246.114.56                      Unit is ready and clustered
   nova-cloud-controller/2               active    idle   2/lxd/6  10.246.114.55   8774/tcp,8775/tcp  Unit is ready
     nova-cloud-controller-hacluster/1   active    idle            10.246.114.55                      Unit is ready and clustered

MySQL 5
~~~~~~~

The percona-cluster charm is used for OpenStack clouds that leverage MySQL 5
software. There is a hybrid aspect to MySQL 5 HA: although the backend is
natively HA, client access demands an external technique be used.

.. important::

   MySQL 5 is used on cloud nodes whose operating system is older than Ubuntu
   20.04 LTS. Percona XtraDB Cluster, based on MySQL 5, is the actual upstream
   source used.

This example will also use the VIP method. These commands will deploy a
three-node MySQL 5 HA active/active cluster, with a VIP of 10.244.40.22. Each
node will reside in a container on existing machines 4, 5, and 6. It is common
to use an application name of 'mysql':

.. code-block:: none

   juju deploy -n 3 --to lxd:4,lxd:5,lxd:6 --config min-cluster-size=3 --config vip=10.244.40.22 percona-cluster mysql
   juju deploy --config cluster_count=3 hacluster mysql-hacluster
   juju add-relation mysql-hacluster:ha mysql:ha

Refer to the `percona-cluster`_ charm README for more information.

MySQL 8
~~~~~~~

MySQL 8 is purely and natively HA; no external technique is necessary.

MySQL 8 always requires at least three database units via the
mysql-innodb-cluster charm. In addition, every OpenStack application requiring
a connection to the database will need its own subordinate mysql-router
application. The latter should be named accordingly at deploy time (e.g.
'<application-name>-mysql-router'). Finally, to connect an OpenStack
application to the database a relation is added between it and the mysql-router
application.

Here is an example of deploying a three-node MySQL 8 cluster. Each node will
reside in a container on existing machines 0, 1, and 2. The cluster will then
be connected to an existing highly available keystone application:

.. code-block:: none

   juju deploy -n 3 --to lxd:0,lxd:1,lxd:2 mysql-innodb-cluster
   juju deploy mysql-router keystone-mysql-router
   juju add-relation keystone-mysql-router:db-router mysql-innodb-cluster:db-router
   juju add-relation keystone-mysql-router:shared-db keystone:shared-db

Below is resulting output from the :command:`juju status` command for such a
scenario:

.. code-block:: console

   Unit                        Workload  Agent  Machine  Public address  Ports     Message
   keystone/6                  active    idle   0/lxd/4  10.246.114.71   5000/tcp  Unit is ready
     keystone-hacluster/0*     active    idle            10.246.114.71             Unit is ready and clustered
     keystone-mysql-router/2   active    idle            10.246.114.71             Unit is ready
   keystone/7*                 active    idle   1/lxd/4  10.246.114.61   5000/tcp  Unit is ready
     keystone-hacluster/1      active    idle            10.246.114.61             Unit is ready and clustered
     keystone-mysql-router/0*  active    idle            10.246.114.61             Unit is ready
   keystone/8                  active    idle   2/lxd/4  10.246.114.72   5000/tcp  Unit is ready
     keystone-hacluster/2      active    idle            10.246.114.72             Unit is ready and clustered
     keystone-mysql-router/1   active    idle            10.246.114.72             Unit is ready
   mysql-innodb-cluster/6*     active    idle   0/lxd/5  10.246.114.58             Unit is ready: Mode: R/W
   mysql-innodb-cluster/7      active    idle   1/lxd/5  10.246.114.59             Unit is ready: Mode: R/O
   mysql-innodb-cluster/8      active    idle   2/lxd/5  10.246.114.60             Unit is ready: Mode: R/O

Scaling out the database cluster can be done in the usual manner (new units
will immediately appear as read-only nodes):

.. code-block:: none

   juju add-unit mysql-innodb-cluster

Refer to the `mysql-router`_ and `mysql-innodb-cluster`_ charm READMEs for more
information.

Ceph
~~~~

High availability in Ceph is achieved by means of a storage node cluster and a
monitor node cluster. As opposed to Swift, Ceph clients connect to storage
nodes (OSD) directly. This is made possible by updated "maps" that are
retrieved from the monitor (MON) cluster.

A three MON node cluster is a typical design whereas a three OSD node cluster
is considered the minimum. Below is one way how such a topology can be created.
Each OSD is deployed to existing machines 7, 8, and 9 and a containerised MON
is placed alongside each OSD:

.. code-block:: none

   juju deploy -n 3 --to 7,8,9 --config osd-devices=/dev/sdb ceph-osd
   juju deploy -n 3 --to lxd:7,lxd:8,lxd:9 --config monitor-count=3 ceph-mon
   juju add-relation ceph-mon:osd ceph-osd:mon

The monitor cluster will not be complete until the specified number of ceph-mon
units (``monitor-count``) have been fully deployed. This is to ensure that a
quorum has been met prior to the initialisation of storage nodes.

.. note::

   The default value of option ``monitor-count`` is '3', but it is best
   practice to provide a value explicitly.

Ceph can support data resilience at the host level or the AZ level (i.e. racks
or groups of racks). Host is the default but the charms can use the Juju
provided AZ information to build a more complex CRUSH map.

Refer to the `ceph-mon charm README`_ and `ceph-osd charm README`_ for more
information.

RabbitMQ
~~~~~~~~

RabbitMQ has native broker clustering; clients can be configured with knowledge
of all units of the cluster and will failover to an alternative unit in the
event that the current selected unit fails. Message queues are also mirrored
between cluster nodes.

A cluster is created simply by deploying multiple application units. This
command will deploy a three-node RabbitMQ HA active/active cluster where the
nodes will be containerised within their respective newly deployed machines.

.. code-block:: none

   juju deploy -n 3 --to lxd,lxd,lxd --config min-cluster-size=3 rabbitmq-server

.. note::

   The default value of option ``cluster-partition-handling`` is 'ignore' as it
   has proven to be the most effective method for dealing with `RabbitMQ
   network partitions`_.

Refer to the `rabbitmq-server`_ charm README for more information.

Swift
~~~~~

Swift is implemented by having storage nodes fronted by a proxy node. Unlike
with Ceph, Swift clients do not communicate directly with the storage nodes but
with the proxy instead. Multiple storage nodes ensure write and read storage
high availability while a cluster of proxy nodes provides HA at the proxy
level. Spanning clusters across geographical regions adds resiliency
(multi-region clusters).

The below example shows one way to deploy a two-node proxy cluster and a
three-node storage cluster, all within a single OpenStack region. The proxy
nodes will be deployed to containers on existing machines 3 and 7 whereas the
storage nodes will be deployed to new machines:

.. code-block:: none

   juju deploy -n 2 --to lxd:3,lxd:7 --config zone-assignment=manual --config replicas=3 swift-proxy
   juju deploy --config zone=1 --config block-device=/dev/sdc swift-storage swift-storage-zone1
   juju deploy --config zone=2 --config block-device=/dev/sdc swift-storage swift-storage-zone2
   juju deploy --config zone=3 --config block-device=/dev/sdc swift-storage swift-storage-zone3

This will result in three storage zones with each zone consisting of a single
storage node, thereby satisfying the replica requirement of three.

.. note::

   The default values for options ``zone-assignment`` and ``replicas`` are
   'manual' and '3' respectively.

Refer to the :doc:`Swift <storage/swift>` page for more information on how to
deploy Swift.

Vault
~~~~~

An HA Vault deployment requires both the etcd and easyrsa applications in
addition to hacluster and MySQL. Also, every vault unit in the cluster must
have its own instance of Vault unsealed.

In these example commands, for simplicity, a single percona-cluster unit is
used:

.. code-block:: none

   juju deploy --to lxd:1 percona-cluster mysql
   juju deploy -n 3 --to lxd:0,lxd:1,lxd:2 --config vip=10.246.114.11 vault
   juju deploy --config cluster_count=3 hacluster vault-hacluster
   juju deploy -n 3 --to lxd:0,lxd:1,lxd:2 etcd
   juju deploy --to lxd:0 cs:~containers/easyrsa
   juju add-relation vault:ha vault-hacluster:ha
   juju add-relation vault:shared-db percona-cluster:shared-db
   juju add-relation etcd:db vault:etcd
   juju add-relation etcd:certificates easyrsa:client

Initialise Vault to obtain the master key shards (KEY-N) and initial root token
(VAULT_TOKEN). Work from an external host that has access to the vault units
and has the ``vault`` snap installed. Do so by referring to any unit
(VAULT_ADDR):

.. code-block:: none

   export VAULT_ADDR="http://10.246.114.58:8200"
   vault operator init -key-shares=5 -key-threshold=3
   export VAULT_TOKEN=s.vhlAKHfkHBvOvRRIE6KIkwRp

Repeat the below command block for each unit. The unit's temporary token used
below is generated by the :command:`token create` subcommand:

.. code-block:: none

   export VAULT_ADDR="http://10.246.114.??:8200"
   vault operator unseal KEY-1
   vault operator unseal KEY-2
   vault operator unseal KEY-3
   vault token create -ttl=10m
   juju run-action --wait vault/leader authorize-charm token=s.ROnC91Y3ByWDDncoZJ3YMtaY

Here is output from the :command:`juju status` command for this deployment:

.. code:: console

   Unit                  Workload  Agent  Machine  Public address  Ports     Message
   easyrsa/0*            active    idle   0/lxd/2  10.246.114.71             Certificate Authority connected.
   etcd/0                active    idle   0/lxd/1  10.246.114.69   2379/tcp  Healthy with 3 known peers
   etcd/1*               active    idle   1/lxd/1  10.246.114.61   2379/tcp  Healthy with 3 known peers
   etcd/2                active    idle   2/lxd/1  10.246.114.70   2379/tcp  Healthy with 3 known peers
   mysql/0*              active    idle   1/lxd/2  10.246.114.72   3306/tcp  Unit is ready
   vault/0               active    idle   0/lxd/0  10.246.114.58   8200/tcp  Unit is ready (active: true, mlock: disabled)
     vault-hacluster/1   active    idle            10.246.114.58             Unit is ready and clustered
   vault/1*              active    idle   1/lxd/0  10.246.114.59   8200/tcp  Unit is ready (active: false, mlock: disabled)
     vault-hacluster/0*  active    idle            10.246.114.59             Unit is ready and clustered
   vault/2               active    idle   2/lxd/0  10.246.114.60   8200/tcp  Unit is ready (active: false, mlock: disabled)
     vault-hacluster/2   active    idle            10.246.114.60             Unit is ready and clustered

Only a single vault unit is active at any given time (reflected in the above
output). The other units will proxy incoming API requests to the active unit
over a secure cluster connection.

Neutron OVS/DVR (legacy)
~~~~~~~~~~~~~~~~~~~~~~~~

Neutron OVS/DVR refers to the traditional functionality of `Open vSwitch`_
(OVS). It may optionally use `Distributed Virtual Routing`_ (DVR) as an
alternate method for creating virtual router topologies. With the advent of OVN
(see below) this framework is regarded as `legacy OpenStack networking`_.

Control plane HA
^^^^^^^^^^^^^^^^

Control plane HA is implemented by the neutron-api and hacluster charms.

Neutron OVS/DVR is configured via the Neutron APIs and maintains its state in
the cloud's database, which has its own HA implementation (see `MySQL 5`_ or
`MySQL 8`_). Workers on the Neutron API nodes respond to requests through
message queues hosted by RabbitMQ, which also has its own HA implementation
(see `RabbitMQ`_).

Data plane HA
^^^^^^^^^^^^^

Data plane HA is implemented by the neutron-gateway and neutron-openvswitch
charms and the post-install network configuration of the cloud.

East/West traffic failures are akin to hypervisor failures: events that cannot
be resolved by HA (but can be mitigated by "instance HA" solutions such as
Masakari). A disruption to North/South traffic however will adversely affect
the entire cloud and can well be prevented through HA.

Data plane HA involves the scheduling of each virtual router to dedicated
gateway nodes (for non-DVR mode) or hypervisors (for DVR mode). Liveness
detection between routers uses a combination of AMQP messaging and the Virtual
Router Redundancy Protocol (VRRP).

In the DVR case, when Floating IPs are used, traffic is handled by the
instance's respective hypervisor. When FIPs are not used a hypervisor is
randomly selected. DVR can therefore render every hypervisor self-sufficient in
terms of routing traffic for its instances. This is a form of HA and is
therefore recommended for clouds that employ Floating IPs. See `High
availability using DVR`_ in the Neutron documentation for more information.

.. note::

   A set of Neutron agents runs on each hypervisor: Open vSwitch agent, DHCP
   agent, and L3 agent. These agents communicate over the RabbitMQ message
   queue with Neutron API workers and any interruption to their services affect
   only their respective hypervisor. There is no HA for these agents but note
   that the components needed for their operation are all HA (RabbitMQ, Neutron
   API, and MySQL).

OVN
~~~

`Open Virtual Network`_ (OVN) complements the existing capabilities of OVS by
adding native support for virtual network abstractions, such as virtual L2 and
L3 overlays and security groups.

.. important::

   OVN is available as an option starting with Ubuntu 20.04 LTS on OpenStack
   Ussuri. The use of OVN obviates the need for the neutron-gateway and
   neutron-openvswitch charms.

Control plane HA
^^^^^^^^^^^^^^^^

The OVN control plane is implemented by the ovn-central charm.

Like Neutron OVS/DVR, the desired state of the system is configured via the
Neutron APIs whose HA is implemented by the hacluster charm. Neutron maintains
its state in the cloud's database, which has its own HA implementation (see
`MySQL 5`_ or `MySQL 8`_). The neutron-api application is made aware of OVN by
means of the neutron-api-plugin-ovn subordinate charm.

The desired state is transferred to an OVN database by Neutron API workers.
The run-time state is the product of having that data translated into a second
OVN database by the ``ovn-northd`` daemon. The daemon, of which there are
multiple copies running and thereby forms its own active/standby cluster, and
its databases are deployed by the ovn-central application. The databases are
configured to use the `OVSDB protocol`_ along with the `Clustered Database
Service Model`_.

The recommended topology is a three-node cluster with the resulting database
cluster uses the `Raft algorithm`_ to ensure consistency. These units, along
with their corresponding ovn-northd services and database cluster, constitute
OVN control plane HA.

Data plane HA
^^^^^^^^^^^^^

The OVN data plane is implemented by the ovn-chassis subordinate charm.

An OVS switch runs on each hypervisor (chassis) and is programmed by the
``ovn-controller`` daemon, which has access to the second (translated) OVN
database.

East/West traffic flows directly from the source chassis to the destination
chassis. North/South traffic passes through gateway chassis that are either
dynamically selected by algorithms or statically configured by the operator;
Floating IPs don't play a special role in that determination.

HA applies to North/South traffic and involves the scheduling of each virtual
router to up to five gateway chassis. Liveness detection between routers is
done using the `BFD protocol`_. East/West traffic disruptions are localised to
individual hypervisors and can be aided by instance HA solutions (e.g.
Masakari).

The recommended topology is to have one ovn-chassis unit placed on each
hypervisor. These units, along with their corresponding ovn-controller daemon,
comprise OVN data plane HA.

Deployment
^^^^^^^^^^

A set of deployment steps for OVN is given below. Specific requisite components
are working nova-compute and vault applications.

.. code-block:: none

   juju deploy neutron-api
   juju deploy neutron-api-plugin-ovn
   juju deploy -n 3 ovn-central
   juju deploy ovn-chassis

   juju add-relation neutron-api-plugin-ovn:certificates vault:certificates
   juju add-relation neutron-api-plugin-ovn:neutron-plugin neutron-api:neutron-plugin-api-subordinate
   juju add-relation neutron-api-plugin-ovn:ovsdb-cms ovn-central:ovsdb-cms
   juju add-relation ovn-central:certificates vault:certificates
   juju add-relation ovn-chassis:ovsdb ovn-central:ovsdb
   juju add-relation ovn-chassis:certificates vault:certificates
   juju add-relation ovn-chassis:nova-compute nova-compute:neutron-plugin

Finally, you will need to provide an SSL certificate. This can be done by
having Vault use a self-signed certificate or by using a certificate chain.
We'll do the former here for simplicity but see :doc:`security/tls` for how to
use a chain.

.. code-block:: none

   juju run-action --wait vault/leader generate-root-ca

Here is select output from the :command:`juju status` command for a minimal
deployment of OVN with MySQL 8:

.. code-block:: console

   Unit                           Workload  Agent  Machine  Public address  Ports              Message
   mysql-innodb-cluster/0*        active    idle   0/lxd/0  10.246.114.61                      Unit is ready: Mode: R/W
   mysql-innodb-cluster/1         active    idle   1/lxd/0  10.246.114.69                      Unit is ready: Mode: R/O
   mysql-innodb-cluster/2         active    idle   2/lxd/0  10.246.114.72                      Unit is ready: Mode: R/O
   neutron-api/0*                 active    idle   3/lxd/1  10.246.114.75   9696/tcp           Unit is ready
     neutron-api-mysql-router/0*  active    idle            10.246.114.75                      Unit is ready
     neutron-api-plugin-ovn/0*    active    idle            10.246.114.75                      Unit is ready
   nova-compute/0*                active    idle   4        10.246.114.58                      Unit is ready
     ovn-chassis/0*               active    idle            10.246.114.58                      Unit is ready
   ovn-central/0*                 active    idle   0/lxd/1  10.246.114.60   6641/tcp,6642/tcp  Unit is ready (leader: ovnsb_db)
   ovn-central/1                  active    idle   1/lxd/1  10.246.114.70   6641/tcp,6642/tcp  Unit is ready (leader: ovnnb_db)
   ovn-central/2                  active    idle   2/lxd/1  10.246.114.71   6641/tcp,6642/tcp  Unit is ready
   vault/0*                       active    idle   3/lxd/2  10.246.114.74   8200/tcp           Unit is ready (active: true, mlock: disabled)
     vault-mysql-router/0*        active    idle            10.246.114.74                      Unit is ready

Refer to the :doc:`networking/ovn` page for more information on how to deploy
OVN.

Other items of interest
-----------------------

Various HA related topics are covered in this section.

Failure detection and alerting
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The detection and alerting of service outages occurring in applications under
HA is especially important. This can take the shape of a full LMA stack but the
essence is the integration of a service application (e.g. keystone) with a
nagios application. These two are joined by means of the `nrpe`_ subordinate
charm. Configuration options available to the service application and to the
nrpe application are used to enable the checks.

Known issues
------------

No major issues at this time.

Consult each charm's bug tracker for full bug listings. See the `OpenStack
Charms`_ project group.

.. LINKS
.. _MAAS: https://maas.io
.. _MAAS zones: https://maas.io/docs/availability-zones
.. _High availability: https://docs.openstack.org/arch-design/arch-requirements/arch-requirements-ha.html
.. _hacluster: https://jaas.ai/hacluster
.. _nrpe: https://jaas.ai/nrpe
.. _OpenStack Charms: https://launchpad.net/openstack-charms
.. _ceph-mon charm README: https://opendev.org/openstack/charm-ceph-mon/src/branch/master/README.md
.. _ceph-osd charm README: https://opendev.org/openstack/charm-ceph-osd/src/branch/master/README.md
.. _ceph-mon: https://jaas.ai/ceph-mon
.. _ceph-osd: https://jaas.ai/ceph-osd
.. _neutron-openvswitch: https://jaas.ai/neutron-openvswitch
.. _nova-compute: https://jaas.ai/nova-compute
.. _neutron-gateway: https://jaas.ai/neutron-gateway
.. _swift-proxy: https://jaas.ai/swift-proxy
.. _swift-storage: https://jaas.ai/swift-storage
.. _ceph-radosgw: https://jaas.ai/ceph-radosgw
.. _mysql-router: https://opendev.org/openstack/charm-mysql-router/src/branch/master/src/README.md
.. _mysql-innodb-cluster: https://opendev.org/openstack/charm-mysql-innodb-cluster/src/branch/master/src/README.md
.. _percona-cluster: https://opendev.org/openstack/charm-percona-cluster/src/branch/master/README.md
.. _rabbitmq-server: https://opendev.org/openstack/charm-rabbitmq-server/src/branch/master/README.md
.. _How to deploy to a specific machine: https://juju.is/docs/olm/deploy-to-a-specific-machine
.. _Bundle reference: https://jaas.ai/docs/bundle-reference
.. _Nova availability zone: https://docs.openstack.org/nova/latest/admin/availability-zones.html
.. _Neutron availability zone: https://docs.openstack.org/neutron/latest/admin/config-az.html
.. _Open Virtual Network: https://docs.openstack.org/networking-ovn/latest/
.. _legacy OpenStack networking: https://docs.openstack.org/liberty/networking-guide/scenario-classic-ovs.html
.. _Open vSwitch: http://www.openvswitch.org
.. _Distributed Virtual Routing: https://wiki.openstack.org/wiki/Neutron/DVR
.. _High availability using DVR: https://docs.openstack.org/neutron/latest/admin/deploy-ovs-ha-dvr.html
.. _RabbitMQ network partitions: https://www.rabbitmq.com/partitions.html
.. _OVSDB protocol: http://docs.openvswitch.org/en/latest/ref/ovsdb.7/#ovsdb
.. _BFD protocol: https://tools.ietf.org/html/rfc5880
.. _Clustered Database Service Model: http://docs.openvswitch.org/en/latest/ref/ovsdb.7/#clustered-database-service-model
.. _Raft algorithm: https://raft.github.io/
.. _Ceph bucket type: https://docs.ceph.com/docs/master/rados/operations/crush-map/#types-and-buckets
.. _Managing power events: https://docs.openstack.org/charm-guide/latest/howto/managing-power-events.html

.. BUGS
.. _LP #1234561: https://bugs.launchpad.net/charm-ceph-osd/+bug/1234561
.. _LP #1882508: https://bugs.launchpad.net/charm-deployment-guide/+bug/1882508
.. _LP #1958674: https://bugs.launchpad.net/charm-nova-cloud-controller/+bug/1958674
