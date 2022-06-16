==========================
Instance high availability
==========================

Overview
--------

As of the 20.05 charm release, Masakari can be deployed to provide automated
instance recovery for clouds that use shared storage for its instances. The
following functionality is provided:

#. **Evacuation of instances** (supported since OpenStack Stein)

   In the event of hypervisor software failure the associated compute node is
   shut down and instance images are started on another hypervisor.

#. **Restarting of instances** (supported since OpenStack Ussuri)

   A failed instance can be restarted on its current hypervisor.

See the `masakari charm`_ for an overview of the charms involved.

.. note::

   `MAAS`_ is required when enabling Masakari on Charmed OpenStack.

Software
--------

Install the software necessary for configuring Masakari:

.. code-block:: none

   sudo snap install openstackclients

Verify that the ``segment`` sub-command is available (this is provided by the
``python-masakariclient`` plugin):

.. code-block:: none

   openstack segment --help

.. important::

   If the ``segment`` sub-command is not available you will need a more recent
   version of the ``openstackclients`` snap. For example, you may need to use
   the 'edge' channel: :command:`sudo snap refresh openstackclients
   --channel=edge`.

Instance evacuation mechanics
-----------------------------

In order for an instance to be relocated to another hypervisor some form of
shared storage must be implemented. As a result, the scenario where a
hypervisor has lost network access to its peers yet continues to access that
shared storage must be considered.

The mechanics of instance evacuation is now described:

Masakari Monitors, on a hypervisor, detects that its peer is unavailable and
notifies the Masakari API server. This in turn triggers the Masakari engine to
initiate a failover of the instance via Nova. Assuming that Nova concurs that
the hypervisor is absent, it will attempt to start the instance on another
hypervisor. At this point there are two instances competing for the same disk
image, which can lead to data corruption.

The solution is to enable a STONITH Pacemaker plugin, which will power off the
compute node via the MAAS API when Pacemaker detects the hypervisor as being
offline.

.. caution::

   Since nova-compute is typically deployed on bare metal, which may host
   containerised applications and possibly even applications alongside
   nova-compute (e.g. ceph-osd), care is advised when designing a cloud with
   Masakari to avoid a powered-off compute node from disrupting crucial non-HA
   cloud services.

   Ensure that Masakari functionality has been fully validated in a staging
   environment prior to using it in production.

Usage
-----

Configuration
~~~~~~~~~~~~~

The below overlay bundle can be used to deploy Masakari when using a bundle to
deploy OpenStack.

Ensure that the ``machines`` section and the placement directives (i.e. the
``to`` option under the masakari application) can co-exist with your OpenStack
bundle.

Provide values for the ``maas_url``, ``maas_credentials``, and ``vip``
hacluster charm options . A VIP is a virtual IP needed for Masakari to enable
HA (a requirement when using the masakari charm). If multiple networks are
used, multiple (space separated) VIPs should be provided. See :doc:`ha` for
HA guidance.

Enable STONITH via the ``enable-stonith`` pacemaker-remote charm option.

Provide values for the ``binding`` (network spaces) masakari charm option
according to your local environment. For simplicity (or for testing), the same
network space can be used for all Masakari bindings.

If the cloud is TLS-enabled via Vault then a relation is needed between the
masakari and vault applications. Uncomment the relation in the overlay bundle
to achieve this. See the :doc:`security/tls` page for background information.

.. important::

   The value for ``openstack-origin`` must match the series and OpenStack
   release of the currently deployed cloud. The value for the global parameter
   ``series`` must also be set accordingly.

.. code-block:: yaml

   machines:
     '0':
     '1':
     '2':
     '3':

   relations:
   - - nova-compute:juju-info
     - masakari-monitors:container
   - - masakari:ha
     - hacluster:ha
   - - keystone:identity-credentials
     - masakari-monitors:identity-credentials
   - - nova-compute:juju-info
     - pacemaker-remote:juju-info
   - - hacluster:pacemaker-remote
     - pacemaker-remote:pacemaker-remote
   - - masakari:identity-service
     - keystone:identity-service
   - - masakari:shared-db
     - mysql:shared-db
   - - masakari:amqp
     - rabbitmq-server:amqp
   #- - vault:certificates
   #  - masakari:certificates

   series: focal

   applications:
     masakari-monitors:
       charm: cs:masakari-monitors
     hacluster:
       charm: cs:hacluster
       options:
         maas_url: <INSERT MAAS URL>
         maas_credentials: <INSERT MAAS API KEY>
     pacemaker-remote:
       charm: cs:pacemaker-remote
       options:
         enable-stonith: True
         enable-resources: False
     masakari:
       charm: cs:masakari
       num_units: 3
       options:
         openstack-origin: cloud:focal-victoria
         vip: <INSERT VIP(S)>
       bindings:
         public: public
         admin: admin
         internal: internal
         shared-db: internal
         amqp: internal
       to:
       - 'lxd:1'
       - 'lxd:2'
       - 'lxd:3'

Deployment
~~~~~~~~~~

To deploy Masakari during the deployment of a new cloud (e.g. via the
`openstack-base`_ bundle):

.. code-block:: none

   juju deploy ./bundle.yaml --overlay masakari-overlay.yaml

To add Masakari to an existing deployment (i.e. the Juju model has pre-existing
machines) the ``--map-machines`` option should be used.

The cloud should then be configured for usage. See
:doc:`cdg:configure-openstack` in the Deploy Guide for assistance.

For the purposes of this document the below hypervisors are presumed:

.. code-block:: console

   +-------------------+---------+-------+
   | Host              | Status  | State |
   +-------------------+---------+-------+
   | virt-node-01.maas | enabled | up    |
   | virt-node-10.maas | enabled | up    |
   | virt-node-02.maas | enabled | up    |
   +-------------------+---------+-------+

In addition let us assume that instance 'focal-1' now resides on host
'virt-node-02.maas':

.. code-block:: console

   +----------------------+-------------------+
   | Field                | Value             |
   +----------------------+-------------------+
   | OS-EXT-SRV-ATTR:host | virt-node-02.maas |
   +----------------------+-------------------+

The above information was obtained by the following two commands,
respectively:

.. code-block:: none

   openstack compute service list -c Host -c Status -c State --service nova-compute
   openstack server show focal-1 -c OS-EXT-SRV-ATTR:host

Instance evacuation recovery methods
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

With Masakari, compute nodes are grouped into failover segments. In the event
of a compute node failure, that node's instances are moved onto another compute
node within the same segment.

The destination node is determined by the recovery method configured for the
affected segment. There are four methods:

* ``reserved_host``
* ``auto``
* ``rh_priority``
* ``auto_priority``

A compute node failure can be simulated by bringing down its primary network
interface. For example, to bring down a node that corresponds to unit
``nova-compute/2``:

.. code-block:: none

   juju run --unit nova-compute/2 sudo ip link set br-ens3 down

'reserved_host'
^^^^^^^^^^^^^^^

The ``reserved_host`` recovery method relocates instances to a subset of
non-active nodes. Because these nodes are not active and are typically
resourced adequately for failover duty, there is a guarantee that sufficient
resources will exist on a reserved node to accommodate migrated instances.

For example, to create segment 'S1', configure it to use the ``reserved_host``
method, and assign it three compute nodes, with one being tagged as a reserved
node:

.. code-block:: none

   openstack segment create S1 reserved_host COMPUTE
   openstack segment host create virt-node-10.maas COMPUTE SSH S1
   openstack segment host create virt-node-02.maas COMPUTE SSH S1
   openstack segment host create --reserved True virt-node-01.maas COMPUTE SSH S1

View the details of a segment:

.. code-block:: none

   openstack segment list

Sample output:

.. code-block:: console

   +--------------------------------------+------+-------------+--------------+-----------------+
   | uuid                                 | name | description | service_type | recovery_method |
   +--------------------------------------+------+-------------+--------------+-----------------+
   | 3af6dfe7-1619-486f-a2c6-8453488c6a66 | S2   | None        | COMPUTE      | auto            |
   +--------------------------------------+------+-------------+--------------+-----------------+

A segment's hosts can be listed like this:

.. code-block:: none

   openstack segment host list -c name -c reserved -c on_maintenance S2

The output should show a value of 'True' in the 'reserved' column for the
appropriate node:

.. code-block:: console

   +-------------------+----------+----------------+
   | name              | reserved | on_maintenance |
   +-------------------+----------+----------------+
   | virt-node-01.maas | True     | False          |
   | virt-node-10.maas | False    | False          |
   | virt-node-02.maas | False    | False          |
   +-------------------+----------+----------------+

Finally, disable the reserved node in Nova so that it becomes non-active, and
thus available for failover:

.. code-block:: none

   openstack compute service set --disable virt-node-01.maas nova-compute

The cloud's compute node list should show a status of 'disabled' for the
appropriate node:

.. code-block:: console

   +-------------------+----------+-------+
   | Host              | Status   | State |
   +-------------------+----------+-------+
   | virt-node-01.maas | disabled | up    |
   | virt-node-10.maas | enabled  | up    |
   | virt-node-02.maas | enabled  | up    |
   +-------------------+----------+-------+

When a compute node failure is detected, Masakari will, in Nova, disable the
failed node and enable a reserved node. The state of the node should also show
as 'down'.

Presuming that node 'virt-node-02.maas' has failed the cloud's compute node
list should become:

.. code-block:: console

   +-------------------+----------+-------+
   | Host              | Status   | State |
   +-------------------+----------+-------+
   | virt-node-01.maas | enabled  | up    |
   | virt-node-10.maas | enabled  | up    |
   | virt-node-02.maas | disabled | down  |
   +-------------------+----------+-------+

The reserved node will begin hosting evacuated instances and Masakari will
remove the reserved flag from it. It will also place the failed node in
maintenance mode.

The segment's host list should show:

.. code-block:: console

   +-------------------+----------+----------------+
   | name              | reserved | on_maintenance |
   +-------------------+----------+----------------+
   | virt-node-01.maas | False    | False          |
   | virt-node-10.maas | False    | False          |
   | virt-node-02.maas | False    | True           |
   +-------------------+----------+----------------+

The expectation is that instance 'focal-1' has been moved from
'virt-node-02.maas' to the reserved node, host 'virt-node-01.maas':

.. code-block:: console

   +----------------------+-------------------+
   | Field                | Value             |
   +----------------------+-------------------+
   | OS-EXT-SRV-ATTR:host | virt-node-01.maas |
   +----------------------+-------------------+

'auto'
^^^^^^

The ``auto`` recovery method relocates instances to any available node in the
same segment. Because all the nodes are active, contrarily to the
``reserved_host`` method, there is no guarantee that sufficient resources will
exist on the destination node to accommodate migrated instances.

For example, to create segment 'S2', configure it to use the ``auto`` method,
and assign it three compute nodes:

.. code-block:: none

   openstack segment create S2 auto COMPUTE
   openstack segment host create virt-node-01.maas COMPUTE SSH S2
   openstack segment host create virt-node-02.maas COMPUTE SSH S2
   openstack segment host create virt-node-10.maas COMPUTE SSH S2

In contrast to the ``reserved_host`` method all the nodes show as active (i.e.
none are reserved):

.. code-block:: console

   +-------------------+----------+----------------+
   | name              | reserved | on_maintenance |
   +-------------------+----------+----------------+
   | virt-node-10.maas | False    | False          |
   | virt-node-02.maas | False    | False          |
   | virt-node-01.maas | False    | False          |
   +-------------------+----------+----------------+

Continuing with the above observation, upon node failure, there are no
hypervisors for Masakari to enable in Nova. A failed node will however be put
``on_maintenance`` in Masakari:

.. code-block:: console

   +-------------------+----------+----------------+
   | name              | reserved | on_maintenance |
   +-------------------+----------+----------------+
   | virt-node-10.maas | False    | False          |
   | virt-node-02.maas | False    | False          |
   | virt-node-01.maas | False    | True           |
   +-------------------+----------+----------------+

'rh_priority' and 'auto_priority'
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The below recovery methods utilise one of the previously described methods but
use the other as a failover.

* ``rh_priority``

  Attempts to evacuate instances using the ``reserved_host`` method. If the
  latter is unsuccessful the ``auto`` method will be used.

* ``auto_priority``

  Attempts to evacuate instances using the ``auto`` method. If the latter is
  unsuccessful the ``reserved_host`` method will be used.

Instance restart
~~~~~~~~~~~~~~~~

The enabling of the instance restart feature is done on a per-instance basis.

For example, tag instance 'focal-1' as HA-enabled in order to have it
restarted automatically on its hypervisor:

.. code-block:: none

   openstack server set --property HA_Enabled=True focal-1

.. important::

   Perhaps non-intuitively, if the instance evacuation feature is not desired a
   hypervisor must nonetheless be assigned a failover segment in order for the
   restart feature to be available to its instances.

An instance failure can be simulated by killing its process. First determine
its hypervisor and ``qemu`` guest name:

.. code-block:: none

   openstack server show focal-1 -c OS-EXT-SRV-ATTR:host -c OS-EXT-SRV-ATTR:instance_name

Output:

.. code-block:: console

   +-------------------------------+-------------------+
   | Field                         | Value             |
   +-------------------------------+-------------------+
   | OS-EXT-SRV-ATTR:host          | virt-node-02.maas |
   | OS-EXT-SRV-ATTR:instance_name | instance-00000001 |
   +-------------------------------+-------------------+

If you do not have admin rights in the cloud the above fields may not be
visible.

This hypervisor corresponds to unit ``nova-compute/2`` in this example cloud.

Check the current PID, kill the process, wait a minute, and verify that a new
process gets started:

.. code-block:: none

   juju run --unit nova-compute/2 'pgrep -f guest=instance-00000001'
   juju run --unit nova-compute/2 'sudo pkill -f -9 guest=instance-00000001'
   juju run --unit nova-compute/2 'pgrep -f guest=instance-00000001'

Supplementary information
-------------------------

This section contains information that can be useful when working with
Masakari.

* Once a failed node has been re-inserted into the cloud it will show, in
  Nova, as 'disabled' but 'up' and, in Masakari, as 'on_maintenance'. It can
  become an active hypervisor with:

  .. code-block:: none

     openstack compute service set --enable <host-name> nova-compute
     openstack segment host update --on_maintenance=False <segment-name> <host-name>

* A segment's recovery method can be updated with:

  .. code-block:: none

     openstack segment update --recovery_method <method> --service_type COMPUTE <segment-name>

* A node cannot be assigned to a segment while it's assigned to another
  segment. It must first be removed from the current segment with:

  .. code-block:: none

     openstack segment host delete <segment-name> <host-name>

* A node's reserved status can be updated with:

  .. code-block:: none

     openstack segment host update --reserved=<boolean> <segment-name> <host-name>

.. LINKS
.. _MAAS: https://maas.io
.. _masakari charm: http://charmhub.io/masakari
.. _openstack-base: https://charmhub.io/openstack-base
