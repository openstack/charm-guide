==========
Nova Cells
==========

.. important::

   This page has been identified as being affected by the breaking changes
   introduced between versions 2.9.x and 3.x of the Juju client. Read
   support note :ref:`juju_29_3x_changes` before continuing.

Overview
++++++++


As of the 18.11 charm release, with OpenStack Rocky and later, multiple nova
cells can be created when deploying a cloud or an existing deployment can be
extended to add additional cells.

Prior to Rocky, and since Pike, all OpenStack charm deployments have two
cells: cell0 which is used to store instances which failed to get scheduled
and cell1 which is an active compute cell.  Additional cells are not supported
by the charms prior to Rocky.

Nova Cells v2 Topology
++++++++++++++++++++++

Nova cells v2 is entirely distinct from Nova cells v1. V1 has been deprecated
and special care should be taken when reading nova documentation to ensure that
it is v2 specific.

.. note::

   This is a *Nova* cell, other services such as Neutron, Glance etc are not
   cell aware.

The Super Conductor
~~~~~~~~~~~~~~~~~~~

For a deployment with multiple cells a dedicated conductor must be run for the
API-level services. This conductor will have access to the API database and a
dedicated message queue. This is called the super conductor to distinguish its
place and purpose from the per-cell conductor nodes
(See `Cells Layout <https://docs.openstack.org/nova/latest/user/cellsv2-layout.html#multiple-cells>`_.). The existing *nova-cloud-controller* charm already performs
the role of the super conductor for the whole cloud and, in addition, acts as a
cell conductor for cell1.


nova-cell-controller charm
~~~~~~~~~~~~~~~~~~~~~~~~~~

A new *nova-cell-controller* charm has been created. This charm just runs the
*nova-conductor* service and proxies some charm information, such as ssh keys,
between the cell compute nodes and the super-conductor. An instance of
*nova-cell-controller* is required per additional cell.

Using application names to distinguish between cells
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Each cell requires the following charms:

* nova-cell-controller
* rabbitmq-server
* mysql-innodb-cluster
* nova-compute

.. note::

   Prior to Ubuntu 20.04 LTS (Focal) the percona-cluster charm (as opposed to
   mysql-innodb-cluster) is used for the cloud database (see
   :doc:`../../project/procedures/percona-series-upgrade-to-focal`). The
   `corresponding page in the Train release`_ of this guide covers
   percona-cluster instructions.

To allow each application to be configured differently between cells and to
be able to distinguish which instance of an application goes with which cell it
is useful to add the cell name to the service name when deploying the
components.

.. code:: bash

   juju deploy nova-compute nova-compute-cell4

Rabbit for Neutron
~~~~~~~~~~~~~~~~~~

One of the motivations for using cells is to help with scaling to a large
number of compute nodes. While not a cells feature (nor a new feature) it is
worth pointing out that the charms support breaking Neutron message traffic
out onto a separate broker.

.. code:: bash

   juju integrate neutron-api:amqp rabbitmq-server-neutron:amqp
   juju integrate neutron-gateway:amqp rabbitmq-server-neutron:amqp
   juju integrate neutron-openvswitch:amqp rabbitmq-server-neutron:amqp
   juju integrate ceilometer:amqp-listener rabbitmq-server-neutron:amqp

Adding A New Cell to an Existing Deployment
+++++++++++++++++++++++++++++++++++++++++++

In an existing deployment cell1 already exists as do the services to support
it. To add 'cell2' to this deployment the new cell applications need to be
deployed and related to the existing applications.

Add the four cell applications:

.. code:: bash

   juju deploy -n 3 ch:mysql-innodb-cluster mysql-innodb-cluster-cell2
   juju deploy ch:mysql-router nova-cell-controller-cell2-mysql-router
   juju deploy ch:rabbitmq-server rabbitmq-server-nova-cell2
   juju deploy ch:nova-compute nova-compute-cell2
   juju deploy ch:nova-cell-controller --config cell-name='cell2' nova-cell-controller-cell2

.. note::

   mysql-innodb-cluster charm deploys HA MySQL database cluster therefore it requires at least three units.

Relate the new cell applications to each other:

.. code:: bash

   juju integrate nova-compute-cell2:amqp rabbitmq-server-nova-cell2:amqp
   juju integrate nova-cell-controller-cell2:amqp rabbitmq-server-nova-cell2:amqp
   juju integrate nova-cell-controller-cell2:shared-db nova-cell-controller-cell2-mysql-router:shared-db
   juju integrate nova-cell-controller-cell2-mysql-router:db-router mysql-innodb-cluster-cell2:db-router
   juju integrate nova-cell-controller-cell2:cloud-compute nova-compute-cell2:cloud-compute

Relate the super conductor to the new cell:

.. code:: bash

   juju integrate nova-cloud-controller:nova-cell-api nova-cell-controller-cell2:nova-cell-compute
   juju integrate nova-cloud-controller:amqp-cell rabbitmq-server-nova-cell2:amqp
   juju integrate nova-cloud-controller:shared-db-cell nova-cell-controller-cell2-mysql-router:shared-db

Relate the new cell to network, image and identity services:

.. code:: bash

   juju integrate nova-compute-cell2:neutron-plugin neutron-openvswitch:neutron-plugin
   juju integrate nova-compute-cell2:image-service glance:image-service
   juju integrate nova-cell-controller-cell2:identity-credentials keystone:identity-credentials
   juju integrate nova-compute-cell2:cloud-credentials keystone:identity-credentials

Relate the new cell to telemetry services.

.. note::

   The ceilometer charm has an *amqp* and an *amqp-listerner* interface.
   ceilometer will listen and post messages to the broker related to the
   *amqp* interface. It will only listen to messages posted to the broker(s)
   related to the *amqp-listener*. Therefore services that consume messages
   from ceilometer, such as aodh, should be related to the broker associated
   with ceilometers *amqp* interface.

.. code:: bash

   juju integrate ceilometer:amqp-listener rabbitmq-server-nova-cell2:amqp
   juju integrate ceilometer-agent:nova-ceilometer nova-compute-cell2:nova-ceilometer

New Deployments
+++++++++++++++

For all cell deployments ensure the following:

* Application naming scheme such that the cell an application belongs to is
  clear.
* Naming the central message broker such that its purpose is clear
  eg rabbitmq-server-general

If cells are being used primarily to help with a large scale out of compute
resources then in addition:

* Do not relate compute nodes to the *nova-cloud-controller*
* Have a separate message broker for Neutron.

Below is an example of an overlay which can be used when doing a fresh deploy
to add a second cell:

.. code:: yaml

   applications:
     mysql-innodb-cluster-cell2:
       charm: ch:mysql-innodb-cluster
       num_units: 3
       options:
         max-connections: 1000
     nova-cell-controller-cell2-mysql-router:
       charm: ch:mysql-router
       num_units: 1
       options:
         base-port: 3316
     nova-cell-controller-cell2:
       charm: ch:nova-cell-controller
       num_units: 1
       options:
         cell-name: "cell2"
     nova-compute-cell2:
       charm: ch:nova-compute
       num_units: 1
       constraints: mem=4G
       options:
         config-flags: default_ephemeral_format=ext4
         enable-live-migration: true
         enable-resize: true
         migration-auth-type: ssh
     rabbitmq-server-nova-cell2:
       charm: ch:rabbitmq-server
       num_units: 1
   relations:
     - - nova-compute-cell2:neutron-plugin
       - neutron-openvswitch:neutron-plugin
     - - nova-compute-cell2:image-service
       - glance:image-service
     - - nova-compute-cell2:cloud-credentials
       - keystone:identity-credentials
     - - nova-cell-controller-cell2:identity-credentials
       - keystone:identity-credentials
     - - nova-cloud-controller:amqp-cell
       - rabbitmq-server-nova-cell2:amqp
     - - nova-cloud-controller:nova-cell-api
       - nova-cell-controller-cell2:nova-cell-compute
     - - nova-cloud-controller:shared-db-cell
       - nova-cell-controller-cell2-mysql-router:shared-db
     - - nova-compute-cell2:amqp
       - rabbitmq-server-nova-cell2:amqp
     - - nova-cell-controller-cell2:amqp
       - rabbitmq-server-nova-cell2:amqp
     - - nova-cell-controller-cell2:shared-db
       - nova-cell-controller-cell2-mysql-router:shared-db
     - - nova-cell-controller-cell2-mysql-router:db-router
       - mysql-innodb-cluster-cell2:db-router
     - - nova-cell-controller-cell2:cloud-compute
       - nova-compute-cell2:cloud-compute
     - - ceilometer:amqp-listener
       - rabbitmq-server-nova-cell2::amqp
     - - ceilometer-agent:nova-ceilometer
       - nova-compute-cell2::nova-ceilometer

Targeting instances at a cell
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Instances can be targeted at a specific cell by manually maintaining host
aggregates and corresponding flavors which target those host aggregates. For
example, assume *cell2* has one compute host *juju-250b86-prod-19*. Create a
host aggregate for *cell2* and add the compute host into it.

.. code:: bash

   openstack aggregate create --property cell=cell2 ag_cell2
   openstack aggregate add host ag_cell2 juju-250b86-prod-19


Now create a flavor that targets that cell.

.. code:: bash

   openstack flavor create --id 5 --ram 2048 --disk 10 --ephemeral 0 --vcpus 1 --public --property cell=cell2 m1.cell2.small

Finally, enable the *AggregateInstanceExtraSpecsFilter*

.. code:: bash

   FILTERS=$(juju config nova-cloud-controller scheduler-default-filters)
   juju config nova-cloud-controller scheduler-default-filters="${FILTERS},AggregateInstanceExtraSpecsFilter"

Now instances that use the *m1.cell2.small* filter will land on cell2 compute
hosts.

.. note::

   These host aggregates need to be manually updated when compute nodes are
   added to the cell.

.. LINKS
.. _corresponding page in the Train release: https://docs.openstack.org/project-deploy-guide/charm-deployment-guide/train/app-nova-cells.html
