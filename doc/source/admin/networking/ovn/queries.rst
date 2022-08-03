============
Querying OVN
============

The OVN databases are configured to use the `Clustered Database Service
Model`_. In this configuration only the leader processes transactions and the
administrative client tools are configured to require a connection to the
leader to operate.

.. note::

   For general information on OVN, refer to the main :doc:`index` page.

The leader of the Northbound and Southbound databases does not have to coincide
with the charm leader, so before querying databases you must consult the output
of :command:`juju status` to check which unit is the leader of the database you
want to query. Example:

.. code-block:: none

   juju status ovn-central

.. code-block:: console

   Unit            Workload  Agent  Machine  Public address  Ports              Message
   ovn-central/0*  active    idle   0/lxd/5  10.246.114.39   6641/tcp,6642/tcp  Unit is ready (leader: ovnnb_db)
   ovn-central/1   active    idle   1/lxd/4  10.246.114.15   6641/tcp,6642/tcp  Unit is ready (northd: active)
   ovn-central/2   active    idle   2/lxd/2  10.246.114.27   6641/tcp,6642/tcp  Unit is ready (leader: ovnsb_db)

In the above example 'ovn-central/0' is the leader for the Northbound DB,
'ovn-central/1' has the active ``ovn-northd`` daemon and 'ovn-central/2' is the
leader for the Southbound DB.

OVSDB Cluster status
--------------------

The cluster status as conveyed through :command:`juju status` is updated each
time a hook is run, in some circumstances it may be necessary to get an
immediate view of the current cluster status.

To get an immediate view of the database clusters:

.. code-block:: none

   juju run --application ovn-central 'ovn-appctl -t \
       /var/run/ovn/ovnnb_db.ctl cluster/status OVN_Northbound'
   juju run --application ovn-central 'ovn-appctl -t \
       /var/run/ovn/ovnsb_db.ctl cluster/status OVN_Southbound'

Querying DBs
------------

To query the individual databases:

.. code-block:: none

   juju run --unit ovn-central/0 'ovn-nbctl show'
   juju run --unit ovn-central/2 'ovn-sbctl show'
   juju run --unit ovn-central/2 'ovn-sbctl lflow-list'

As an alternative you may provide the administrative client tools with
command-line arguments for path to certificates and IP address of servers so
that you can run the client from anywhere:

.. code-block:: none

   ovn-nbctl \
      -p /etc/ovn/key_host \
      -C /etc/ovn/ovn-central.crt \
      -c /etc/ovn/cert_host \
      --db ssl:10.246.114.39:6641,ssl:10.246.114.15:6641,ssl:10.246.114.27:6641 \
      show

Note that for remote administrative write access to the Southbound DB you must
use port number '16642'. This is due to OVN RBAC being enabled on the standard
'6642' port:

.. code-block:: none

   ovn-sbctl \
      -p /etc/ovn/key_host \
      -C /etc/ovn/ovn-central.crt \
      -c /etc/ovn/cert_host \
      --db ssl:10.246.114.39:16642,ssl:10.246.114.15:16642,ssl:10.246.114.27:16642 \
      show

Data plane flow tracing
-----------------------

Connect (by SSH) to one of the chassis units to get access to various
diagnostic tools:

.. code-block:: none

   juju ssh ovn-chassis/0

   sudo ovs-vsctl show

   sudo ovs-ofctl -O OpenFlow13 dump-flows br-int

   sudo ovs-appctl -t ovs-vswitchd \
      ofproto/trace br-provider \
      in_port=enp3s0f0,icmp,nw_src=192.0.2.1,nw_dst=192.0.2.100

   sudo ovn-trace \
      -p /etc/ovn/key_host \
      -C /etc/ovn/ovn-chassis.crt \
      -c /etc/ovn/cert_host \
      --db ssl:10.246.114.39:6642,ssl:10.246.114.15:6642,ssl:10.246.114.27:6642 \
      --ovs ext-net 'inport=="provnet-dde76bc9-0620-44f7-b99a-99cfc66e1095" && \
      eth.src==30:e1:71:5c:7a:b5 && \
      eth.dst==fa:16:3e:f7:15:73 && \
      ip4.src==10.172.193.250 && \
      ip4.dst==10.246.119.8 && \
      icmp4.type==8 && \
      ip.ttl == 64'

.. note::

   OVN makes use of OpenFlow 1.3 (and newer) and as such the charm configures
   bridges to use these protocols. To be able to successfully use the
   :command:`ovs-ofctl` command you must specify the OpenFlow version as shown
   in the example above.

   You may issue the :command:`ovs-vsctl list bridge` command to show what
   protocols are enabled on the bridges.

.. LINKS
.. _Clustered Database Service Model: http://docs.openvswitch.org/en/latest/ref/ovsdb.7/#clustered-database-service-model
