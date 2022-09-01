=====================
Managing power events
=====================

Overview
--------

Once your OpenStack cloud is deployed and in production you will need to
consider how to manage applications in terms of shutting them down and starting
them up. Examples of situations where this knowledge would be useful include
controlled power events such as node reboots and restarting an AZ (or an entire
cloud). You will also be better able to counter uncontrolled power events like
a power outage. This guide covers how to manage these kinds of power events in
your cloud successfully.

For the purposes of this document, a *node* is any non-containerised system
that houses at least one cloud service. In practice, this typically constitutes
a physical host.

In addition, any `known issues`_ affecting the restarting of parts of the cloud
stack are documented. Although they are presented last, it is highly
recommended to review them prior to attempting to apply any of the information
shown here.

An important assumption made in this document is that the cloud is
*hyperconverged*. That is, multiple applications cohabit each cloud node. This
aspect makes a power event especially significant as it can potentially affect
the entire cloud.

.. note::

   This document may help influence a cloud's initial design. Once it
   is understood how an application should be treated in the context
   of a power event the cloud architect will be able to make better
   informed decisions.

Section `Notable applications`_ contains valuable information when stopping and
starting services. It will be used in the context of power events but its
contents can also be used during the normal operation of a cloud.

General guidelines
------------------

As each cloud is unique this section will provide general guidelines on how to
prepare for and manage power events in your cloud.

.. important::

   It is recommended that every deployed cloud have a list of detailed
   procedures that cover the uniqueness of that cloud. The guidelines
   in this current document can act as starting point for such a
   resource.

HA applications
~~~~~~~~~~~~~~~

Theoretically, an application with high availability is resilient to a power
event, meaning that such an event would have no impact on both client requests
to the application and the application itself. However, depending on the
situation, some such applications may still require attention when starting
back up. The `percona-cluster`_ and `mysql-innodb-cluster`_ applications are
good examples of this.

Cloud applications are typically made highly available through the use of the
`hacluster`_ subordinate charm. Some applications, though, achieve HA at the
software layer (outside of Juju), and can be called *natively HA*. One such
application is ``rabbitmq-server``. See :doc:`ha` for more information.

Cloud topology
~~~~~~~~~~~~~~

The very first step is to map out the topology of your cloud. In other words,
you need to know what application units are running on what machines, and
whether those machines are physical (metal), virtual (kvm), or container (lxd)
in nature. Each application's HA status should also be indicated.

A natural way for Juju operators to map out their cloud is by inspecting the
output of the ``juju status`` command. For a demonstration see :ref:`Cloud
topology example <cloud_topology_example>`. It is based on this production
:ref:`Reference cloud <reference_cloud>`.

Control plane, data plane, and shutdown order
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

*Data plane* services involve networking, storage, and virtualisation, whereas
*control plane* services are necessary to administer and operate the cloud.
See `High availability`_ and `Control plane architecture`_ for more details.

When a cloud is in production the priority for the administrator is to ensure
that instances and their associated workloads continue to run. This means that
in terms of the impact a power event may have, the data plane has priority
over the control plane.

Generally, data plane services (DP) are stopped prior to control plane (CP)
services. Also, services within a plane will typically depend upon another
service within that same plane. The conclusion is that the dependant service
should be brought down before the service being depended upon (e.g. stop Nova
before stopping Ceph).

In terms of core applications then, an approximate service shutdown ordered
list can be built to act as a general guideline. Some services, such as API
services, have less, if any, impact on other services and can therefore be
turned off in any order.

In the below list, the most notable aspects are the extremes: nova-compute and
Ceph should be stopped first and keystone, rabbitmq-server, and percona-cluster
(or mysql-innodb-cluster) should be stopped last:

#. ``nova-compute`` (DP)
#. ``ceph-osd`` (DP)
#. ``ceph-mon`` (DP)
#. ``ceph-radosgw`` (DP)
#. ``neutron-gateway`` (DP)
#. ``neutron-openvswitch`` (DP)
#. ``glance`` (CP)
#. ``cinder`` (CP)
#. ``neutron-gateway`` (CP)
#. ``neutron-api`` (CP)
#. ``placement`` (CP)
#. ``nova-cloud-controller`` (CP)
#. ``keystone`` (CP)
#. ``rabbitmq-server`` (CP)
#. ``percona-cluster`` or ``mysql-innodb-cluster`` (CP)

Each node can now be analysed to see what applications it hosts and in what
order they should be stopped.

Stopping and starting services
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When **stopping** a service (not an entire application and not a unit agent) on
a hyperconverged cloud node it is safer to act on each unit and stop the
service individually. The alternative is to power down the node hosting the
service, which will, of course, stop every other service hosted on that node.
**Ensure that you understand the consequences of powering down a node**.

In addition, whenever a service is stopped on a node you need to know what
impact that will have on the cloud. For instance, the default effect of turning
off a Ceph OSD is that data will be re-distributed among the other OSDs,
resulting in high disk and network activity. Most services should be in HA mode
but you should be aware of the quorum that must be maintained in order for HA
to function as designed. For example, turning off two out of three Keystone
cluster members is not advisable.

Wherever possible, this document shows how to manage services with Juju
`actions`_. Apart from their intrinsic benefits (i.e. sanctioned by experts),
actions are not hampered by SSH-restricted environments. Note that a charm may
not implement every desired command in the form of an action however. In that
case, the only alternative is to interact directly with the unit's operating
system via `SSH`_.

.. important::

   When an action is used the resulting state persists within Juju, and, in
   particular, will **survive a node reboot**. This can be very advantageous in
   the context of controlled shutdown and startup procedures, but it does
   demand tracking on the part of the operator. To assist with this, some
   charms expose action information in the output of the ``juju status``
   command .

When actions are **not** used, in terms of **starting** services on a single
node or across a cloud, it may not be possible to do so in a prescribed order
unless the services were explicitly configured to *not* start automatically
during the bootup of a node.

.. QUESTION

   pmatulis: It is possible to start (and stop) LXD containers in a
   certain order. Is adding this element to bundles a viable response
   to the above for LXD-based workloads?`

Regardless of whether a service is started with a Juju action, via SSH, or by
booting the corresponding node, it is vital that you verify afterwards that the
service is actually running and functioning properly.

Controlled power events
-----------------------

The heart of managing your cloud in terms of controlled power events is the
power-cycling of an individual cloud node. Once you're able to make decisions
on a per-node basis extending the power event to a group of nodes, such as an
AZ or even an entire cloud, will become less daunting.

Power-cycling a cloud node
~~~~~~~~~~~~~~~~~~~~~~~~~~

When a hyperconverged cloud node requires to be power-cycled begin by
considering the cloud topology, at least for the machine in question.

To illustrate, machines **17**, **18**, **20** from the :ref:`Cloud topology
example <cloud_topology_example>` will be used. Note that only fundamental
applications will be included (i.e. applications such as openstack-dashboard,
ceilometer, etc. will be omitted).

The main issue behind power-cycling a node is to come up with a **shutdown**
list of services, as the startup list is typically just the shutdown list in
reverse. This is what is shown below for each machine. Information regarding HA
status and machine type has been retained (from the source topology example).

The shutdown lists are based on section `Control plane, data plane, and
shutdown order`_.

machine 17
^^^^^^^^^^

#. ``nova-compute`` (metal)
#. ``ceph-osd`` (natively HA; metal)
#. ``ceph-mon`` (natively HA; lxd)
#. ``ceph-radosgw`` (natively HA; lxd)
#. ``glance`` (HA; lxd)
#. ``cinder`` (HA; lxd)
#. ``keystone`` (HA; lxd)
#. ``percona-cluster`` (HA; lxd)

machine 18
^^^^^^^^^^

#. ``nova-compute`` (metal)
#. ``ceph-osd`` (natively HA; metal)
#. ``neutron-api`` (HA; lxd)
#. ``nova-cloud-controller`` (HA; lxd)
#. ``rabbitmq-server`` (natively HA; lxd)

machine 20
^^^^^^^^^^

#. ``ceph-osd`` (natively HA; metal)
#. ``neutron-gateway`` (natively HA; metal)
#. ``neutron-api`` (HA; lxd)
#. ``nova-cloud-controller`` (HA; lxd)
#. ``rabbitmq-server`` (natively HA; lxd)

See section `Notable applications`_ for instructions on stopping individual
services.

Power-cycling an AZ or an entire cloud
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Apart from the difference in scale of the service outage, stopping and starting
an AZ (availability zone) or an entire cloud is a superset of the case of
power-cycling an individual node. You just need to identify the group of nodes
that are involved. An AZ or cloud would consist of all of the core services
listed in section `Control plane, data plane, and shutdown order`_.

Uncontrolled power events
-------------------------

In the context of this document, an uncontrolled power event is an unintended
power outage. The result of such an event is that one or many physical cloud
hosts have turned off non-gracefully. Since we now know that some cloud
services should be stopped in a particular order and in a particular way the
task now is to ascertain what services could have been negatively impacted and
how to proceed in getting such services back in working order.

Begin as was done in the case of `Power-cycling a cloud node`_ by determining
the topology of the affected nodes. See whether any corresponding services have
special shutdown procedures as documented in section `Notable applications`_.
Any such services will require special scrutiny when they are eventually
started. Determine an ordered startup list for the affected services. As was
shown in `Power-cycling a cloud node`_, this list is the reverse of the
shutdown list. Finally, once the nodes are powered on, by abiding as much as
possible to the startup list, act on any verification steps found in section
`Notable applications`_ for all cloud services.

.. important::

   To prevent affected machines from turning back on automatically, and thus
   interfering with the startup procedures for your cloud, it is recommended to
   disable the auto-poweron BIOS setting on all cloud nodes.

Notable applications
--------------------

This section contains application-specific shutdown/restart procedures,
well-known caveats, or just valuable tips.

As noted under `Stopping and starting services`_, this document encourages the
use of actions for managing application services. The general syntax is::

    juju run-action --wait <unit> <action>

In the procedures that follow, <unit> will be replaced by an example only (e.g.
``nova-compute/0``). You will need to substitute in the actual unit for your
cloud.

For convenience, the applications are listed here (you can also use the table
of contents in the upper left-hand-side):

+-----------------+--------------+-------------------------+--------------------------+
| `ceph-osd`_     | `etcd`_      | `mysql-innodb-cluster`_ | `nova-cloud-controller`_ |
+-----------------+--------------+-------------------------+--------------------------+
| `ceph-mon`_     | `glance`_    | `neutron-gateway`_      | `percona-cluster`_       |
+-----------------+--------------+-------------------------+--------------------------+
| `ceph-radosgw`_ | `keystone`_  | `neutron-openvswitch`_  | `rabbitmq-server`_       |
+-----------------+--------------+-------------------------+--------------------------+
| `cinder`_       | `landscape`_ | `nova-compute`_         | `vault`_                 |
+-----------------+-----------+----------------------------+--------------------------+

-------------------------------------------------------------------------------

.. _ceph-osd:
.. _ceph-mon:
.. _ceph-radosgw:

ceph
~~~~

All Ceph services are grouped under this one heading.

.. note::

   Some ceph-related charms are lacking in actions. Some procedures will
   involve direct intervention. See bugs `LP #1846049`_, `LP #1846050`_, `LP
   #1849222`_, and `LP #1849224`_.

shutdown
^^^^^^^^

With respect to powering down a node that hosts an OSD, by default, the Ceph
CRUSH map is configured to treat each cluster machine as a failure domain. The
default pool behaviour is to replicate data across three failure domains, and
require at least two of them to be present to accept writes. Shutting down
multiple machines too quickly may cause two of three copies of a particular
placement group to become temporarily unavailable, which would cause consuming
applications to block on writes. The CRUSH map can be configured to spread
replicas over a failure domain other than machines. See `CRUSH maps`_ in the
Ceph documentation.

The shutdown procedures for Ceph are provided for both a **cluster** and for
individual **components** (e.g. ``ceph-mon``).

cluster
"""""""

1. Ensure that the cluster is in a healthy state. From a Juju client, run a
   status check on any MON unit::

    juju ssh ceph-mon/1 sudo ceph status

2. Shut down all components/clients consuming Ceph before shutting down Ceph
   components to avoid application-level data loss.

3. Set the cluster-wide ``noout`` option, on any MON unit, to prevent data
   rebalancing from occurring when OSDs start disappearing from the network::

    juju run-action --wait ceph-mon/1 set-noout

   Query status again to ensure that the option is set::

    juju ssh ceph-mon/1 sudo ceph status

   Expected partial output is::

    health: HEALTH_WARN
    noout flag(s) set

4. Stop the RADOS Gateway service on **each** ``ceph-radosgw`` unit.

   First get the current status::

    juju ssh ceph-radosgw/0 systemctl status ceph-radosgw@\*

   Example partial output is::

    ● ceph-radosgw@rgw.ip-172-31-93-254.service - Ceph rados gateway
       Loaded: loaded (/lib/systemd/system/ceph-radosgw@.service; indirect; vendor
       preset: enabled)
          Active: active (running) since Mon 2019-09-30 21:33:53 UTC; 9min ago

   Now pause the service::

    juju run-action --wait ceph-radosgw/0 pause

   Verify that the service has stopped::

    juju ssh ceph-radosgw/0 systemctl status ceph-radosgw@\*

   Expected output is null (no output).

5. Stop all of a unit's OSDs. Do this on **each** ``ceph-osd`` unit::

    juju run-action --wait ceph-osd/1 stop osds=all

   Once done, verify that all of the cluster's OSDs are down::

    juju ssh ceph-mon/1 sudo ceph status

   Assuming a total of six OSDs, expected partial output ("0 up") is::

    osd: 6 osds: 0 up, 6 in; 66 remapped pgs

6. Stop the MON service on **each** ``ceph-mon`` unit::

    juju ssh ceph-mon/0 sudo systemctl stop ceph-mon.service

   Verify that the MON service has stopped on each unit::

    juju ssh ceph-mon/0 systemctl status ceph-mon.service

   Expected partial output is::

    Active: inactive (dead) since Mon 2019-09-30 19:46:09 UTC; 1h 1min ago

.. important::

   Once the MON units have lost quorum you will lose the ability to
   query the cluster.

component
"""""""""

1. Ensure that the cluster is in a healthy state. On any MON::

    juju ssh ceph-mon/1 sudo ceph status

2. **ceph-mon** - To bring down a single MON service:

   a. Stop the MON service on the ``ceph-mon`` unit::

       juju ssh ceph-mon/0 sudo systemctl stop ceph-mon.service

   b. Do not bring down another MON until the cluster has recovered from the
      loss of the current one (run a status check).

3. **ceph-osd** - To take 'out' a single OSD:

   a. Mark the OSD (with id 2) on a ``ceph-osd`` unit as 'out'::

       juju run-action --wait ceph-osd/2 osd-out osds=2

   b. Do not mark OSDs on another unit as 'out' until the cluster has recovered
      from the loss of the current one (run a status check).

4. **ceph-osd** - To stop a single OSD:

   Mark the OSD (with id 2) on a ``ceph-osd`` unit as 'down'::

    juju run-action --wait ceph-osd/2 stop osds=2

5. **ceph-osd** - To take 'out' all the OSDs on a single unit:

   a. Mark all the OSDs on a ``ceph-osd`` unit as 'out'::

       juju run-action --wait ceph-osd/2 osd-out osds=all

   b. Do not mark OSDs on another unit as 'out' until the cluster has recovered
      from the loss of the current ones (run a status check).

6. **ceph-osd** - To stop all the OSDs on a single unit:

   Mark all the OSDs on a ``ceph-osd`` unit as 'down'::

    juju run-action --wait ceph-osd/2 stop osds=all

startup
^^^^^^^

The startup procedures for Ceph are provided for both a **cluster** and for
individual **components** (e.g. ``ceph-mon``).

cluster
"""""""

Nodes hosting Ceph services should be powered on such that the services are
started in this order:

1. ``ceph-mon``
2. ``ceph-osd``
3. ``ceph-radosgw``

**Important**: If during cluster shutdown,

a. the ``noout`` option was set, you will need to unset it. On any MON unit::

    juju run-action --wait ceph-mon/0 unset-noout

b. a RADOS Gateway service was paused, you will need to resume it. Do this for
   **each** ``ceph-radosgw`` unit::

    juju run-action --wait ceph-radosgw/0 resume

Finally, ensure that the cluster is in a healthy state by running a status
check on any MON unit::

    juju ssh ceph-mon/0 sudo ceph status

component
"""""""""

1. Ensure that the cluster is in a healthy state. On any MON::

    juju ssh ceph-mon/0 sudo ceph status

2. **ceph-mon** - To start a single MON service:

   a. Start the MON service on the ``ceph-mon`` unit::

       juju ssh ceph-mon/1 sudo systemctl start ceph-mon.service

   b. Do not bring up another MON until the cluster has recovered from the
      addition of the current one (run a status check).

3. **ceph-osd** - To set as 'in' a single OSD on a unit:

   a. Re-insert the OSD (with id 2) on the ``ceph-osd`` unit::

       juju run-action --wait ceph-osd/1 osd-in osds=2

4. **ceph-osd** - To set as 'in' all the OSDs on a unit:

   a. Re-insert the OSDs on the ``ceph-osd`` unit::

       juju run-action --wait ceph-osd/1 osd-in osds=all

   b. Do not re-insert OSDs on another unit until the cluster has recovered
      from the addition of the current ones (run a status check).

-------------------------------------------------------------------------------

cinder
~~~~~~

shutdown
^^^^^^^^

To pause the Cinder service::

    juju run-action --wait cinder/0 pause

startup
^^^^^^^

To resume the Cinder service::

    juju run-action --wait cinder/0 resume

-------------------------------------------------------------------------------

etcd
~~~~

.. note::

   The ``etcd`` charm is lacking in actions. Some procedures will
   involve direct intervention. See bug `LP #1846257`_.

shutdown
^^^^^^^^

To stop the Etcd service::

    juju ssh etcd/0 sudo systemctl stop snap.etcd.etcd

startup
^^^^^^^

To start the Etcd service::

    juju ssh etcd/0 sudo systemctl start snap.etcd.etcd

read queries
^^^^^^^^^^^^

To see the etcd cluster status. On any ``etcd`` unit::

    juju run-action --wait etcd/0 health

loss of etcd quorum
^^^^^^^^^^^^^^^^^^^

If the majority of the etcd units fail (e.g. 2 out of 3) you can scale down the
cluster (e.g. 3 to 1). However, if all hooks have not had a chance to run (e.g.
you may have to force remove and redeploy faulty units) the surviving master
will not accept new cluster members/units. In that case, do the following:

1. Scale down the cluster to 1 unit any way you can (remove faulty units / stop
   the etcd service / delete the database on the slave units).

2. Force the surviving master to become a 1-node cluster. On the appropriate
   unit:

   a. Stop the service::

       juju ssh etcd/0 sudo systemctl stop snap.etcd.etcd

   b. Connect to the unit via SSH and edit
      ``/var/snap/etcd/common/etcd.conf.yml`` by setting ``force-new-cluster``
      to 'true'.

   c. Start the service::

       juju ssh etcd/0 sudo systemctl start snap.etcd.etcd

   d. Connect to the unit via SSH and edit
      ``/var/snap/etcd/common/etcd.conf.yml`` by setting ``force-new-cluster``
      to 'false'.

3. Scale up the cluster by adding new etcd units.

-------------------------------------------------------------------------------

glance
~~~~~~

shutdown
^^^^^^^^

To pause the Glance service::

    juju run-action --wait glance/0 pause

.. important::

   If Glance is clustered using the 'hacluster' charm, first **pause**
   hacluster and then **pause** Glance.

startup
^^^^^^^

To resume the Glance service::

    juju run-action --wait glance/0 resume

.. important::

   If Glance is clustered using the 'hacluster' charm, first
   **resume** Glance and then **resume** hacluster.

-------------------------------------------------------------------------------

keystone
~~~~~~~~

shutdown
^^^^^^^^

To pause the Keystone service::

    juju run-action --wait keystone/0 pause

.. important::

   If Keystone is clustered using the 'hacluster' charm, first
   **pause** hacluster and then **pause** Keystone.

startup
^^^^^^^

To resume the Keystone service::

    juju run-action --wait keystone/0 resume

.. important::

   If Keystone is clustered using the 'hacluster' charm, first
   **resume** Keystone and then **resume** hacluster.

-------------------------------------------------------------------------------

landscape
~~~~~~~~~

.. note::

   The ``postgresql`` charm, needed by Landscape, is lacking in
   actions. Some procedures will involve direct intervention. See bug
   `LP #1846279`_.

shutdown
^^^^^^^^

1. Pause the Landscape service::

    juju run-action --wait landscape-server/0 pause

2. Stop the PostgreSQL service::

    juju ssh postgresql/0 sudo systemctl stop postgresql

3. Pause the RabbitMQ service::

    juju run-action --wait rabbitmq-server/0 pause

.. caution::

   Services other than Landscape may also be using either of the
   PostgreSQL or RabbitMQ services.

startup
^^^^^^^

The startup of Landscape should be done in the reverse order.

1. Ensure the RabbitMQ service is started::

    juju run-action --wait rabbitmq-server/0 resume

2. Ensure the PostgreSQL service is started::

    juju ssh postgresql/0 sudo systemctl start postgresql

3. Resume the Landscape service::

    juju run-action --wait landscape-server/0 resume

-------------------------------------------------------------------------------

mysql-innodb-cluster
~~~~~~~~~~~~~~~~~~~~

shutdown
^^^^^^^^

To pause the MySQL InnoDB Cluster for a mysql-innodb-cluster unit:

.. code-block:: none

   juju run-action --wait mysql-innodb-cluster/0 pause

To gracefully shut down the cluster repeat the above for every unit.

.. _mysql_innodb_cluster_startup:

startup
^^^^^^^

A special startup procedure is necessary regardless of how services were shut
down (gracefully, hard shutdown, or power outage).

Upon startup the cluster will need to be initialised. It is recommended to read
the upstream document `Rebooting a Cluster from a Major Outage`_ before
proceeding.

At this time the output to command :command:`juju status mysql-innodb-cluster`
should look similar to:

.. code-block:: console

   App                   Version  Status   Scale  Charm                 Store       Channel  Rev  OS      Message
   mysql-innodb-cluster  8.0.25   blocked      3  mysql-innodb-cluster  charmstore  stable     7  ubuntu  Cluster is inaccessible from this instance. Please check logs for details.

   Unit                     Workload  Agent  Machine  Public address  Ports  Message
   mysql-innodb-cluster/0   blocked   idle   0/lxd/2  10.0.0.240             Cluster is inaccessible from this instance. Please check logs for details.
   mysql-innodb-cluster/1   blocked   idle   1/lxd/2  10.0.0.208             Cluster is inaccessible from this instance. Please check logs for details.
   mysql-innodb-cluster/2*  blocked   idle   2/lxd/2  10.0.0.218             Cluster is inaccessible from this instance. Please check logs for details.

Determine the GTID node
"""""""""""""""""""""""

A Juju action needs to be run on the mysql-innodb-cluster unit that corresponds
to the cluster member that possesses the GTID superset (of transactions). This
is the unit that is most up-to-date in terms of cluster activity. The GTID node
therefore needs to be determined. In practice however, it is much easier to
simply run the action on any unit and, if it is the incorrect unit, have it
report which unit does have the GTID. This is the method that will be used
here.

Initialise the cluster
""""""""""""""""""""""

Initialise the cluster by running the ``reboot-cluster-from-complete-outage``
action on any unit:

.. code-block:: none

   juju run-action --wait mysql-innodb-cluster/1 reboot-cluster-from-complete-outage

Here we see, in the command's partial output, that the chosen unit does not
correspond to the GTID node:

.. code-block:: console

   RuntimeError: Dba.reboot_cluster_from_complete_outage: The active session instance (10.0.0.208:3306)
   isn't the most updated in comparison with the ONLINE instances of the Cluster's metadata.
   Please use the most up to date instance: '10.0.0.218:3306'.

This says that the GTID node has an IP address of 10.0.0.218. For us, this
corresponds to unit ``mysql-innodb-cluster/2``. Therefore:

.. code-block:: none

   juju run-action --wait mysql-innodb-cluster/2 reboot-cluster-from-complete-outage

This time, the output should include:

.. code-block:: console

   results:
     outcome: Success
     output: ""
   status: completed

The mysql-innodb-cluster application should now be back to a clustered and
healthy state:

.. code-block:: console

   App                   Version  Status  Scale  Charm                 Store       Channel  Rev  OS      Message
   mysql-innodb-cluster  8.0.25   active      3  mysql-innodb-cluster  charmstore  stable     7  ubuntu  Unit is ready: Mode: R/O, Cluster is ONLINE and can tolerate up to ONE failure.

   Unit                     Workload  Agent  Machine  Public address  Ports  Message
   mysql-innodb-cluster/0   active    idle   0/lxd/2  10.0.0.240             Unit is ready: Mode: R/O, Cluster is ONLINE and can tolerate up to ONE failure.
   mysql-innodb-cluster/1   active    idle   1/lxd/2  10.0.0.208             Unit is ready: Mode: R/O, Cluster is ONLINE and can tolerate up to ONE failure.
   mysql-innodb-cluster/2*  active    idle   2/lxd/2  10.0.0.218             Unit is ready: Mode: R/W, Cluster is ONLINE and can tolerate up to ONE failure.

-------------------------------------------------------------------------------

neutron-gateway
~~~~~~~~~~~~~~~

neutron agents
^^^^^^^^^^^^^^

A cloud outage will occur if a node hosting a non-HA ``neutron-gateway`` is
power cycled due to the lack of neutron agents.

Before stopping the service you can manually check for HA status of neutron
agents on the node using the commands below. HA is confirmed by the presence of
more than one agent per **router**, in the case of L3 agents, and more than one
per **network**, in the case of DHCP agents.

To return the list of **L3 agents** serving each of the routers connected to a
node:

.. code-block:: none

   for i in `openstack network agent list | grep L3 | awk '/$NODE/ {print $2}'` ; \
   do printf "\nAgent $i serves:" ; \
       for f in `neutron router-list-on-l3-agent $i | awk '/network_id/ {print$2}'` ; \
       do printf "\n Router $f served by these agents:\n" ; \
           neutron l3-agent-list-hosting-router $f ; \
       done ; \
   done

To return the list of **DHCP agents** serving each of the networks connected to
a node:

.. code-block:: none

   for i in `openstack network agent list| grep -i dhcp |  awk '/$NODE/ {print $2}'` ; \
   do printf "\nAgent $i serves:" ; \
       for f in `neutron net-list-on-dhcp-agent $i | awk '!/+/ {print$2}'` ; \
       do printf "\nNetwork $f served by these agents:\n" ; \
           neutron dhcp-agent-list-hosting-net $f ; \
       done ; \
   done

.. note::

   Replace ``$NODE`` with the node hostname as known to OpenStack
   (i.e. ``openstack host list``).

shutdown
^^^^^^^^

To pause a Neutron gateway service::

    juju run-action --wait neutron-gateway/0 pause

startup
^^^^^^^

To resume a Neutron gateway service::

    juju run-action --wait neutron-gateway/0 resume

-------------------------------------------------------------------------------

neutron-openvswitch
~~~~~~~~~~~~~~~~~~~

shutdown
^^^^^^^^

To pause the Open vSwitch service::

    juju run-action --wait neutron-openvswitch/0 pause

startup
^^^^^^^

To resume the Open vSwitch service::

    juju run-action --wait neutron-openvswitch/0 resume

-------------------------------------------------------------------------------

nova-cloud-controller
~~~~~~~~~~~~~~~~~~~~~

shutdown
^^^^^^^^

To pause Nova controller services (Nova scheduler, Nova api, Nova network, Nova
objectstore)::

    juju run-action --wait nova-cloud-controller/0 pause

startup
^^^^^^^

To resume Nova controller services::

    juju run-action --wait nova-cloud-controller/0 resume

-------------------------------------------------------------------------------

nova-compute
~~~~~~~~~~~~

.. _nova-compute-shutdown:

shutdown
^^^^^^^^

True HA is not possible for ``nova-compute`` nor its instances. If a node
hosting this service is power-cycled the corresponding hypervisor is removed
from the pool of available hypervisors, and its instances will become
inaccessible. Generally speaking, individual hypervisors are fallible
components in a cloud. The standard response to this is to implement HA on the
instance workloads. Provided shared storage is set up, you can also move
instances to another compute node and boot them anew (state is lost) - see
`Evacuate instances`_.

To stop a Nova service:

1. Some affected nova instances may require a special shutdown sequence (e.g.
   an instance may host a workload that demands particular care when turning it
   off). Invoke them now.

2. Gracefully stop all remaining affected nova instances.

3. Pause the Nova service::

    juju run-action --wait nova-compute/0 pause

.. tip::

   If shared storage is implemented, instead of shutting down
   instances you may consider moving ("evacuating") them to another
   compute node. See `Evacuate instances`_.

startup
^^^^^^^

To resume a Nova service::

    juju run-action --wait nova-compute/0 resume

Instances that fail to come up properly can be moved to another compute host
(see `Evacuate instances`_).

-------------------------------------------------------------------------------

percona-cluster
~~~~~~~~~~~~~~~

shutdown
^^^^^^^^

To pause the Percona XtraDB service for a ``percona-cluster`` unit:

.. code-block:: none

   juju run-action --wait percona-cluster/0 pause

To gracefully shut down the cluster repeat the above for every unit.

startup
^^^^^^^

A special startup procedure is necessary regardless of how services were shut
down (gracefully, hard shutdown, or power outage).

Upon startup the cluster will be in a state described by either scenario 3 or 6
in the upstream document `How to recover a PXC cluster`_. The latter
documentation provides important context to the steps outlined below.

Both scenarios will require a unit to be assigned the role of "bootstrap node".

.. warning::

   Data loss may occur if an incorrect bootstrap node is chosen.

The steps will also involve the concept of application unit leadership. An
application leader unit is denoted by an asterisk in the Unit column of the
:command:`juju status` output.

Determine the bootstrap node
""""""""""""""""""""""""""""

Determine the bootstrap node by examining `Percona XtraDB sequence numbers`_.
The percona-cluster units either have the same sequence number or they do not.
Sequence numbers are displayed in the output of the :command:`juju status`
command.

.. note::

   Alternatively, the sequence number can be found on the corresponding
   machine's filesystem in file
   ``/var/lib/percona-xtradb-cluster/grastate.dat``.

Example #1 - Same sequence number

In this output the units have a common sequence number of '355'. This indicates
that any unit can act as the bootstrap node:

.. code-block:: console

   Unit                Workload  Agent  Machine  Public address  Ports     Message
   keystone/0*         active    idle   0        10.5.0.32       5000/tcp  Unit is ready
   percona-cluster/0   blocked   idle   1        10.5.0.20       3306/tcp  MySQL is down. Sequence Number: 355. Safe To Bootstrap: 0
     hacluster/0       active    idle            10.5.0.20                 Unit is ready and clustered
   percona-cluster/1   blocked   idle   2        10.5.0.17       3306/tcp  MySQL is down. Sequence Number: 355. Safe To Bootstrap: 0
     hacluster/1       active    idle            10.5.0.17                 Unit is ready and clustered
   percona-cluster/2*  blocked   idle   3        10.5.0.27       3306/tcp  MySQL is down. Sequence Number: 355. Safe To Bootstrap: 0
     hacluster/2*      active    idle            10.5.0.27                 Unit is ready and clustered

Example #2 - Different sequence numbers

In this output the units do not have a common sequence number. **The unit
chosen as the bootstrap node must be the one with the greatest sequence
number.** Here it is unit ``percona-cluster/2``, with a number of '1325':

.. code-block:: console

   Unit                Workload  Agent  Machine  Public address  Ports     Message
   keystone/0*         active    idle   0        10.5.0.32       5000/tcp  Unit is ready
   percona-cluster/0*  blocked   idle   1        10.5.0.20       3306/tcp  MySQL is down. Sequence Number: 1318. Safe To Bootstrap: 0
     hacluster/0*      active    idle            10.5.0.20                 Unit is ready and clustered
   percona-cluster/1   blocked   idle   2        10.5.0.17       3306/tcp  MySQL is down. Sequence Number: 1318. Safe To Bootstrap: 0
     hacluster/1       active    idle            10.5.0.17                 Unit is ready and clustered
   percona-cluster/2   blocked   idle   3        10.5.0.27       3306/tcp  MySQL is down. Sequence Number: 1325. Safe To Bootstrap: 0
     hacluster/2       active    idle            10.5.0.27                 Unit is ready and clustered

Initialise the cluster
""""""""""""""""""""""

Initialise the cluster by running the ``bootstrap-pxc`` action on the chosen
bootstrap node unit. In this example it is ``percona-cluster/2``, which happens
to be a non-leader.

.. code-block:: none

   juju run-action --wait percona-cluster/2 bootstrap-pxc

Notify the cluster of the new bootstrap UUID
""""""""""""""""""""""""""""""""""""""""""""

The cluster will typically require being notified of the new "bootstrap UUID".

In the vast majority of cases, once the ``bootstrap-pxc`` action has been run,
and the model has settled, the output to the :command:`juju status` command
will look like this:

.. code-block:: console

   Unit                Workload  Agent  Machine  Public address  Ports     Message
   keystone/0*         active    idle   0        10.5.0.32       5000/tcp  Unit is ready
   percona-cluster/0*  waiting   idle   1        10.5.0.20       3306/tcp  Unit waiting for cluster bootstrap
     hacluster/0*      active    idle            10.5.0.20                 Unit is ready and clustered
   percona-cluster/1   waiting   idle   2        10.5.0.17       3306/tcp  Unit waiting for cluster bootstrap
     hacluster/1       active    idle            10.5.0.17                 Unit is ready and clustered
   percona-cluster/2   waiting   idle   3        10.5.0.27       3306/tcp  Unit waiting for cluster bootstrap
     hacluster/2       active    idle            10.5.0.27                 Unit is ready and clustered

The message "Unit waiting for cluster bootstrap" indicates that the cluster
needs to be notified of the new bootstrap UUID, and is done via the
``notify-bootstrapped`` action. Which unit to apply this action against depends
on how the previous action was used:

#. If ``bootstrap-pxc`` was run on the leader then ``notify-bootstrapped``
   must be run on a non-leader.
#. Inversely, if ``bootstrap-pxc`` was run on a non-leader then
   ``notify-bootstrapped`` must be run on the leader.

In the current example, the first action was run on a non-leader
(``percona-cluster/2``). The second action should therefore be run on the
leader, which here is ``percona-cluster/0``:

.. code-block:: none

   juju run-action --wait percona-cluster/0 notify-bootstrapped

After the model settles, the status output should show all nodes in active and
ready state:

.. code-block:: console

   Unit                Workload  Agent  Machine  Public address  Ports     Message
   keystone/0*         active    idle   0        10.5.0.32       5000/tcp  Unit is ready
   percona-cluster/0*  active    idle   1        10.5.0.20       3306/tcp  Unit is ready
     hacluster/0*      active    idle            10.5.0.20                 Unit is ready and clustered
   percona-cluster/1   active    idle   2        10.5.0.17       3306/tcp  Unit is ready
     hacluster/1       active    idle            10.5.0.17                 Unit is ready and clustered
   percona-cluster/2   active    idle   3        10.5.0.27       3306/tcp  Unit is ready
     hacluster/2       active    idle            10.5.0.27                 Unit is ready and clustered

The percona-cluster application is now back to a clustered and healthy state.

-------------------------------------------------------------------------------

rabbitmq-server
~~~~~~~~~~~~~~~

shutdown
^^^^^^^^

To pause a RabbitMQ service::

    juju run-action --wait rabbitmq-server/0 pause

startup
^^^^^^^

To resume a RabbitMQ service::

    juju run-action --wait rabbitmq-server/0 resume

read queries
^^^^^^^^^^^^

Provided rabbitmq is running on a ``rabbitmq-server`` unit, you can perform a
status check::

    juju run-action --wait rabbitmq-server/1 cluster-status

Example partial output is:

.. code-block:: console

   Cluster status of node 'rabbit@ip-172-31-13-243'
    [{nodes,[{disc,['rabbit@ip-172-31-13-243']}]},
     {running_nodes,['rabbit@ip-172-31-13-243']},
     {cluster_name,<<"rabbit@ip-172-31-13-243.ec2.internal">>},
     {partitions,[]},
     {alarms,[{'rabbit@ip-172-31-13-243',[]}]}]

It is expected that there are no objects listed on the partitions line (as
above).

To list unconsumed queues (those with pending messages)::

    juju run-action --wait rabbitmq-server/1 list-unconsumed-queues

See `Partitions`_ and `Queues`_ in the RabbitMQ documentation.

partitions
^^^^^^^^^^

Any partitioned units will need to be attended to. Stop and start the
rabbitmq-server service for each ``rabbitmq-server`` unit, checking for status
along the way:

.. code-block:: none

   juju run-action --wait rabbitmq-server/0 pause
   juju run-action --wait rabbitmq-server/1 cluster-status
   juju run-action --wait rabbitmq-server/0 pause
   juju run-action --wait rabbitmq-server/1 cluster-status

If errors persist, the mnesia database will need to be removed from the
affected unit so it can be resynced from the other units. Do this by removing
the contents of the ``/var/lib/rabbitmq/mnesia`` directory between the stop and
start commands.

.. note::

    The network partitioning handling mode configured by the
    ``rabbitmq-server`` charm is ``autoheal``.

cluster startup problems
^^^^^^^^^^^^^^^^^^^^^^^^

By design, the last cluster node to shut down (primary broker) needs
to be the first one to start up. If the primary broker is not
available when restarting RabbitMQ units after an abnormal shut down
such as during a power loss or a scheduled cluster restart the
secondary RabbitMQ units will try to contact the primary broker for 5
minutes before giving up to start the cluster. This condition can be
identified by log entries such as

.. code-block:: console

   =WARNING REPORT==== 27-Apr-2021::19:50:45 ===
   Error while waiting for Mnesia tables: {timeout_waiting_for_tables,
                                           [rabbit_user,rabbit_user_permission,
                                            rabbit_vhost,rabbit_durable_route,
                                            rabbit_durable_exchange,
                                            rabbit_runtime_parameters,
                                            rabbit_durable_queue]}

   =INFO REPORT==== 27-Apr-2021::19:50:45 ===
   Waiting for Mnesia tables for 30000 ms, 0 retries left

   =INFO REPORT==== 27-Apr-2021::19:51:21 ===
   Timeout contacting cluster nodes: ['rabbit@juju-766a10-14',
                                      'rabbit@juju-766a10-15'].

Using the above log entries as an example, ``rabbit@juju-766a10-16``
tried to synchronize its queues with ``rabbit@juju-766a10-14`` and
``rabbit@juju-766a10-15`` but could not reach either. The cluster is
not operational.

In order to recover from this scenario both offline units,
``rabbit@juju-766a10-14`` and ``rabbit@juju-766a10-15``, need be to
started and unit ``rabbit@juju-766a10-16`` either rebooted or its
``rabbitmq-server`` service restarted once the other 2 units are
available.

It should be verified with

.. code-block:: none

   sudo rabbitmqctl cluster_status

on one of the running units that the RabbitMQ cluster is operational.
In case a unit gets stuck in an ``error`` state the command

.. code-block:: none

   juju resolve rabbitmq-server/0

can be used to clear this status.

If the primary broker is not available the cluster can be forced to
start by running the ``force-boot`` action on the remaining units,
e.g.

.. code-block:: none

   juju run-action --wait rabbitmq-server/0 force-boot

which makes use of the RabbitMQ `force_boot`_ option. The cluster will
become operational, however, it will be running on fewer units and
will not offer the same high availability and scalability. Either add
another unit or bring the missing primary broker online in order to
restore the cluster.

.. note::

   The ``force-boot`` action may cause the loss of queue data. See the
   upstream documentation on `Restarting Cluster Nodes`_ for more
   details.

.. note::

   Every unit or broker in a RabbitMQ cluster is equivalent and the
   term ``primary broker`` only applies in a shutdown scenario
   referring to the last broker to go down.

-------------------------------------------------------------------------------

vault
~~~~~

With HA Vault, each unit may need to be processed individually.

.. note::

   The vault charm is lacking in actions. Some procedures will involve direct
   intervention. See bug `LP #1846282`_.

.. warning::

   Ensure that the unseal keys are available before pausing a vault unit.

shutdown
^^^^^^^^

To pause a Vault service::

    juju run-action --wait vault/0 pause

The :command:`juju status` command will return: ``blocked, Vault service not
running``.

startup
^^^^^^^

To resume a Vault service::

    juju run-action --wait vault/0 resume

The :command:`juju status` command will return: ``blocked, Unit is sealed``.

read queries
^^^^^^^^^^^^

To see Vault service status::

    juju ssh vault/0 /snap/bin/vault status

Expected output is::

    Cluster is sealed

unsealing units
^^^^^^^^^^^^^^^

The unit will manually (and locally) need to be unsealed with its respective
``VAULT_ADDR`` environment variable and with the minimum number of unseal keys
(three here):

.. code-block:: none

   export VAULT_ADDR="http://<IP of vault unit>:8200"
   vault operator unseal <key>
   vault operator unseal <key>
   vault operator unseal <key>

Once the model has settled, the :command:`juju status` command will return:
``active, Unit is ready...``

Known issues
------------

- `LP #1804261`_ : ceph-osds will need to be restarted if they start before Vault is ready and unsealed
- `LP #1818260`_ : forget cluster node failed during cluster-relation-changed hook
- `LP #1818680`_ : booting should succeed even if vault is unavailable
- `LP #1818973`_ : vault fails to start when MySQL backend down
- `LP #1827690`_ : barbican-worker is down: Requested revision 1a0c2cdafb38 overlaps with other requested revisions 39cf2e645cba
- `LP #1840706`_ : install hook fails with psycopg2 ImportError

Consult each charm's bug tracker for full bug listings. See the `OpenStack
Charms`_ project group.

.. LINKS
.. _percona-cluster charm: https://opendev.org/openstack/charm-percona-cluster/src/branch/master/README.md#cold-boot
.. _How to recover a PXC cluster: https://www.percona.com/blog/2014/09/01/galera-replication-how-to-recover-a-pxc-cluster
.. _Percona XtraDB sequence numbers: https://www.percona.com/blog/2017/12/14/sequence-numbers-seqno-percona-xtradb-cluster/
.. _High availability: https://docs.openstack.org/arch-design/arch-requirements/arch-requirements-ha.html
.. _Control plane architecture: https://docs.openstack.org/arch-design/design-control-plane.html
.. _Evacuate instances: https://docs.openstack.org/nova/latest/admin/evacuate.html
.. _hacluster: https://jaas.ai/hacluster
.. _OpenStack Charms: https://launchpad.net/openstack-charms
.. _SSH: https://juju.is/docs/olm/accessing-individual-machines-with-ssh
.. _CRUSH maps: https://docs.ceph.com/docs/master/rados/operations/crush-map
.. _actions: https://juju.is/docs/olm/working-with-actions
.. _Partitions: https://www.rabbitmq.com/partitions.html
.. _Queues: https://www.rabbitmq.com/queues.html
.. _force_boot: https://www.rabbitmq.com/rabbitmqctl.8.html#force_boot
.. _Restarting Cluster Nodes: https://www.rabbitmq.com/clustering.html#restarting
.. _Rebooting a Cluster from a Major Outage: https://dev.mysql.com/doc/mysql-shell/8.0/en/troubleshooting-innodb-cluster.html#reboot-outage

.. BUGS
.. _LP #1804261: https://bugs.launchpad.net/charm-ceph-osd/+bug/1804261
.. _LP #1818260: https://bugs.launchpad.net/charm-rabbitmq-server/+bug/1818260
.. _LP #1818680: https://bugs.launchpad.net/charm-ceph-osd/+bug/1818680
.. _LP #1818973: https://bugs.launchpad.net/vault-charm/+bug/1818973
.. _LP #1827690: https://bugs.launchpad.net/charm-barbican/+bug/1827690
.. _LP #1840706: https://bugs.launchpad.net/vault-charm/+bug/1840706
.. _LP #1846049: https://bugs.launchpad.net/charm-ceph-mon/+bug/1846049
.. _LP #1846050: https://bugs.launchpad.net/charm-ceph-mon/+bug/1846050
.. _LP #1846257: https://bugs.launchpad.net/charm-etcd/+bug/1846257
.. _LP #1846279: https://bugs.launchpad.net/postgresql-charm/+bug/1846279
.. _LP #1846282: https://bugs.launchpad.net/vault-charm/+bug/1846282
.. _LP #1846375: https://bugs.launchpad.net/vault-charm/+bug/1846375
.. _LP #1849222: https://bugs.launchpad.net/charm-ceph-mon/+bug/1849222
.. _LP #1849224: https://bugs.launchpad.net/charm-ceph-radosgw/+bug/1849224
