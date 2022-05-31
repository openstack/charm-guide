=====
Swift
=====

Overview
--------

There are two fundamental ways to deploy a Swift cluster with charms. Each
method differs in how storage nodes get assigned to storage zones.

As of the 20.02 charm release, with OpenStack Newton or later, support for a
multi-region (global) cluster, which is an extension of the single-region
scenario, is available. See the upstream documentation on `Global clusters`_
for background information.

.. warning::

   Charmed Swift global cluster functionality is in a preview state and is
   ready for testing. It is not production-ready.

Any Swift deployment relies upon the `swift-proxy`_ and `swift-storage`_
charms. Refer to those charms to learn about the various configuration options
used throughout this guide.

Single-region cluster
---------------------

Manual zone assignment
~~~~~~~~~~~~~~~~~~~~~~

The 'manual' method (the default) allows the cluster to be designed by
explicitly assigning a storage zone to a storage node. This zone gets
associated with the swift-storage application. This means that this method
involves multiple uniquely-named swift-storage applications.

Let file ``swift.yaml`` contain the configuration:

.. code-block:: yaml

   swift-proxy:
       zone-assignment: manual
       replicas: 3
   swift-storage-zone1:
       zone: 1
       block-device: /dev/sdb
   swift-storage-zone2:
       zone: 2
       block-device: /dev/sdb
   swift-storage-zone3:
       zone: 3
       block-device: /dev/sdb

Deploy the proxy and storage nodes:

.. code-block:: none

   juju deploy --config swift.yaml swift-proxy
   juju deploy --config swift.yaml swift-storage swift-storage-zone1
   juju deploy --config swift.yaml swift-storage swift-storage-zone2
   juju deploy --config swift.yaml swift-storage swift-storage-zone3

Add relations between the proxy node and all storage nodes:

.. code-block:: none

   juju add-relation swift-proxy:swift-storage swift-storage-zone1:swift-storage
   juju add-relation swift-proxy:swift-storage swift-storage-zone2:swift-storage
   juju add-relation swift-proxy:swift-storage swift-storage-zone3:swift-storage

This will result in a three-zone cluster, with each zone consisting of a single
storage node, thereby satisfying the replica requirement of three.

Storage capacity is increased by adding swift-storage units to a zone. For
example, to add two storage nodes to zone '3':

.. code-block:: none

   juju add-unit -n 2 swift-storage-zone3

.. note::

   When scaling out ensure the candidate machines are equipped with the block
   devices currently configured for the associated application.

This charm will not balance the storage ring until there are enough storage
zones to meet its minimum replica requirement, in this case three.

Auto zone assignment
~~~~~~~~~~~~~~~~~~~~

The 'auto' method automatically assigns storage zones to storage nodes. There
is only one swift-storage application and only one relation between it and the
swift-proxy application. The relation sets up the initial storage node in zone
'1' (by default). Newly-added nodes get assigned to zone '2', '3', and so on,
until the number of stated replicas is reached. Zone numbering then falls back
to '1' and the zone-assignment cycle repeats. In this way, storage nodes get
distributed evenly across zones.

Let file ``swift.yaml`` contain the configuration:

.. code-block:: yaml

   swift-proxy:
       zone-assignment: auto
       replicas: 3
   swift-storage:
       block-device: /dev/sdb

Deploy the proxy node and the storage application:

.. code-block:: none

   juju deploy --config swift.yaml swift-proxy
   juju deploy --config swift.yaml swift-storage

The first storage node gets assigned to zone '1' when the initial relation is
added:

.. code-block:: none

   juju add-relation swift-proxy:swift-storage swift-storage:swift-storage

The second and third units get assigned to zones '2' and '3', respectively,
during scale-out operations:

.. code-block:: none

   juju add-unit -n 2 swift-storage

.. note::

   When scaling out ensure the candidate machines are equipped with the block
   devices currently configured for the associated application.

At this point the replica requirement is satisfied and the ring is balanced.
The ring is extended by continuing to add more units to the single application.

Multi-region cluster
--------------------

The previous configurations provided a single-region cluster. Generally a
region is composed of a group of nodes with high-bandwidth, low-latency links
between them. This almost always translates to the same geographical location.

Multiple such clusters can be meshed together to create a multi-region (global)
cluster. The goal is to achieve greater data resiliency by spanning zones
across geographically dispersed regions.

This section includes two configurations for implementing a Swift global
cluster: minimal and comprehensive.

A global cluster is an extension of the single cluster scenario. Refer to
the `Single-region cluster`_ section for information on essential options.

Minimal configuration
~~~~~~~~~~~~~~~~~~~~~

The proxy and storage nodes for a global cluster require extra configuration:

On the proxy node,

* option ``enable-multi-region`` is set to 'true'
* option ``region`` is defined
* option ``swift-hash`` is defined (same value for all regions)

On the storage nodes,

* option ``storage-region`` is set

The below example has two storage regions, a single zone, one storage node per
storage region, and a replica requirement of two. Manual zone assignment will
be used.

Let file ``swift.yaml`` contain the configuration:

.. code-block:: yaml

   swift-proxy-region1:
       region: RegionOne
       zone-assignment: manual
       replicas: 2
       enable-multi-region: true
       swift-hash: "efcf2102-b9e9-4d71-afe6-000000111111"
   swift-proxy-region2:
       region: RegionTwo
       zone-assignment: manual
       replicas: 2
       enable-multi-region: true
       swift-hash: "efcf2102-b9e9-4d71-afe6-000000111111"
   swift-storage-region1:
       storage-region: 1
       zone: 1
       block-device: /dev/sdb
   swift-storage-region2:
       storage-region: 2
       zone: 1
       block-device: /dev/sdb

The value of ``swift-hash`` is arbitrary. It is provided here in the form of a
UUID.

.. important::

   The name of a storage region must be an integer. Here, OpenStack region
   'RegionOne' corresponds to storage region '1', and OpenStack region
   'RegionTwo' corresponds to storage region '2'.

Deploy in RegionOne:

.. code-block:: none

   juju deploy --config swift.yaml swift-proxy swift-proxy-region1
   juju deploy --config swift.yaml swift-storage swift-storage-region1

Deploy in RegionTwo:

.. code-block:: none

   juju deploy --config swift.yaml swift-proxy swift-proxy-region2
   juju deploy --config swift.yaml swift-storage swift-storage-region2

Add relations between swift-proxy in RegionOne and swift-storage in both
RegionOne and RegionTwo:

.. code-block:: none

   juju add-relation swift-proxy-region1:swift-storage swift-storage-region1:swift-storage
   juju add-relation swift-proxy-region1:swift-storage swift-storage-region2:swift-storage

Add relations between swift-proxy in RegionTwo and swift-storage in both
RegionOne and RegionTwo:

.. code-block:: none

   juju add-relation swift-proxy-region2:swift-storage swift-storage-region1:swift-storage
   juju add-relation swift-proxy-region2:swift-storage swift-storage-region2:swift-storage

Add a relation between swift-proxy in RegionOne and swift-proxy in RegionTwo:

.. code-block:: none

   juju add-relation swift-proxy-region1:rings-distributor swift-proxy-region2:rings-consumer

More than one proxy can be deployed per OpenStack region, and each must have a
relation to every proxy in all other OpenStack regions. Only one proxy can act
as a "rings-distributor" at any one time; the proxy in RegionOne was
arbitrarily chosen.

Comprehensive configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~

A global cluster is primarily useful when there are groups of storage nodes and
proxy nodes in different physical regions, creating a
geographically-distributed cluster. These regions typically reside in distinct
Juju models, making `Cross model relations`_ a necessity. In addition, there
are configuration options available for tuning read and write behaviour. The
next example demonstrates how to implement these features and options in a
realistic scenario.

Refer to the `Minimal configuration`_ section for basic settings.

Tuning configuration
^^^^^^^^^^^^^^^^^^^^

The ``read-affinity`` option is used to control what order the regions and
zones are examined when searching for an object. A common approach would be to
put the local region first on the search path for a proxy. For instance, in the
deployment example below the Swift proxy in New York is configured to read from
the New York storage nodes first. Similarly the San Francisco proxy prefers
storage nodes in San Francisco.

The ``write-affinity`` option allows nodes to be stored locally before being
eventually distributed globally. This write_affinity setting is useful only
when you do not read objects immediately after writing them.

The ``write-affinity-node-count`` option is used to further configure
``write-affinity``. This option dictates how many local storage servers will be
tried before falling back to remote ones.

Storage regions are referred to by prepending an 'r' to their names. Hence 'r1'
refers to storage region '1'. Similarly for zones. Zone '1' is referred to by
'z1'.

For more details on these options see the upstream `Global clusters`_ document.

Deployment
^^^^^^^^^^

.. warning::

   Throughout this guide make sure ``openstack-origin`` matches the value you
   used when `deploying OpenStack`_.

This example assumes there are two data centres, one in San Francisco (SF) and
one in New York (NY). These contain Juju models 'swift-sf' and 'swift-ny'
respectively. Model 'swift-ny' contains OpenStack region 'RegionOne' and
storage region '1'. Model 'swift-sf' contains OpenStack region 'RegionTwo' and
storage region '2'.

Bundle overlays are needed for encapsulating cross-model relations. So the
deployment in each OpenStack region consists of both a bundle and an overlay.

This is the contents of bundle ``swift-ny.yaml``:

.. code-block:: yaml

   series: bionic
   applications:
     swift-proxy-region1:
       charm: cs:swift-proxy
       num_units: 1
       options:
         region: RegionOne
         zone-assignment: manual
         replicas: 2
         enable-multi-region: true
         swift-hash: "efcf2102-b9e9-4d71-afe6-000000111111"
         read-affinity: "r1=100, r2=200"
         write-affinity: "r1, r2"
         write-affinity-node-count: '1'
         openstack-origin: cloud:bionic-train
     swift-storage-region1-zone1:
       charm: cs:swift-storage
       num_units: 1
       options:
         storage-region: 1
         zone: 1
         block-device: /etc/swift/storage.img|2G
         openstack-origin: cloud:bionic-train
     swift-storage-region1-zone2:
       charm: cs:swift-storage
       num_units: 1
       options:
         storage-region: 1
         zone: 2
         block-device: /etc/swift/storage.img|2G
         openstack-origin: cloud:bionic-train
     swift-storage-region1-zone3:
       charm: cs:swift-storage
       num_units: 1
       options:
         storage-region: 1
         zone: 3
         block-device: /etc/swift/storage.img|2G
         openstack-origin: cloud:bionic-train
     percona-cluster:
       charm: cs:percona-cluster
       num_units: 1
       options:
         dataset-size: 25%
         max-connections: 1000
         source: cloud:bionic-train
     keystone:
       expose: True
       charm: cs:keystone
       num_units: 1
       options:
         openstack-origin: cloud:bionic-train
     glance:
       expose: True
       charm: cs:glance
       num_units: 1
       options:
         openstack-origin: cloud:bionic-train
   relations:
     - - swift-proxy-region1:swift-storage
       - swift-storage-region1-zone1:swift-storage
     - - swift-proxy-region1:swift-storage
       - swift-storage-region1-zone2:swift-storage
     - - swift-proxy-region1:swift-storage
       - swift-storage-region1-zone3:swift-storage
     - - keystone:shared-db
       - percona-cluster:shared-db
     - - glance:shared-db
       - percona-cluster:shared-db
     - - glance:identity-service
       - keystone:identity-service
     - - swift-proxy-region1:identity-service
       - keystone:identity-service
     - - glance:object-store
       - swift-proxy-region1:object-store

This is the contents of overlay bundle ``swift-ny-offers.yaml``:

.. code-block:: yaml

   applications:
     keystone:
       offers:
         keystone-offer:
           endpoints:
           - identity-service
     swift-proxy-region1:
       offers:
         swift-proxy-region1-offer:
           endpoints:
           - swift-storage
           - rings-distributor
     swift-storage-region1-zone1:
       offers:
         swift-storage-region1-zone1-offer:
           endpoints:
           - swift-storage
     swift-storage-region1-zone2:
       offers:
         swift-storage-region1-zone2-offer:
           endpoints:
           - swift-storage
     swift-storage-region1-zone3:
       offers:
         swift-storage-region1-zone3-offer:
           endpoints:
           - swift-storage

This is the contents of bundle ``swift-sf.yaml``:

.. code-block:: yaml

   series: bionic
   applications:
     swift-proxy-region2:
       charm: cs:swift-proxy
       num_units: 1
       options:
         region: RegionTwo
         zone-assignment: manual
         replicas: 2
         enable-multi-region: true
         swift-hash: "efcf2102-b9e9-4d71-afe6-000000111111"
         read-affinity: "r1=100, r2=200"
         write-affinity: "r1, r2"
         write-affinity-node-count: '1'
         openstack-origin: cloud:bionic-train
     swift-storage-region2-zone1:
       charm: cs:swift-storage
       num_units: 1
       options:
         storage-region: 2
         zone: 1
         block-device: /etc/swift/storage.img|2G
         openstack-origin: cloud:bionic-train
     swift-storage-region2-zone2:
       charm: cs:swift-storage
       num_units: 1
       options:
         storage-region: 2
         zone: 2
         block-device: /etc/swift/storage.img|2G
         openstack-origin: cloud:bionic-train
     swift-storage-region2-zone3:
       charm: cs:swift-storage
       num_units: 1
       options:
         storage-region: 2
         zone: 3
         block-device: /etc/swift/storage.img|2G
         openstack-origin: cloud:bionic-train
   relations:
     - - swift-proxy-region2:swift-storage
       - swift-storage-region2-zone1:swift-storage
     - - swift-proxy-region2:swift-storage
       - swift-storage-region2-zone2:swift-storage
     - - swift-proxy-region2:swift-storage
       - swift-storage-region2-zone3:swift-storage

This is the contents of overlay bundle ``swift-sf-consumer.yaml``:

.. code-block:: yaml

   relations:
   - - swift-proxy-region2:identity-service
     - keystone:identity-service
   - - swift-proxy-region2:swift-storage
     - swift-storage-region1-zone1:swift-storage
   - - swift-proxy-region2:swift-storage
     - swift-storage-region1-zone2:swift-storage
   - - swift-proxy-region2:swift-storage
     - swift-storage-region1-zone3:swift-storage
   - - swift-storage-region2-zone1:swift-storage
     - swift-proxy-region1:swift-storage
   - - swift-storage-region2-zone2:swift-storage
     - swift-proxy-region1:swift-storage
   - - swift-storage-region2-zone3:swift-storage
     - swift-proxy-region1:swift-storage
   - - swift-proxy-region2:rings-consumer
     - swift-proxy-region1:rings-distributor
   saas:
     keystone:
       url: admin/swift-ny.keystone-offer
     swift-proxy-region1:
       url: admin/swift-ny.swift-proxy-region1-offer
     swift-storage-region1-zone1:
       url: admin/swift-ny.swift-storage-region1-zone1-offer
     swift-storage-region1-zone2:
       url: admin/swift-ny.swift-storage-region1-zone2-offer
     swift-storage-region1-zone3:
       url: admin/swift-ny.swift-storage-region1-zone3-offer

With the current configuration, ``swift-ny.yaml`` must be deployed first as it
contains the Juju "offers" that ``swift-sf.yaml`` will consume:

.. code-block:: none

   juju deploy -m swift-ny ./swift-ny.yaml --overlay ./swift-ny-offers.yaml
   juju deploy -m swift-sf ./swift-sf.yaml --overlay ./swift-sf-consumer.yaml

.. LINKS
.. _deploying OpenStack: install-openstack
.. _Global clusters: https://docs.openstack.org/swift/latest/overview_global_cluster.html
.. _Cross model relations: https://juju.is/docs/olm/cross-model-relations
.. _swift-proxy: https://jaas.ai/swift-proxy
.. _swift-storage: https://jaas.ai/swift-storage
