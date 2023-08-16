:orphan:

======================================================
Replace a hyperconverged Ceph storage and compute node
======================================================

.. important::

   This page has been identified as being affected by the breaking changes
   introduced between versions 2.9.x and 3.x of the Juju client. Read
   support note :ref:`juju_29_3x_changes` before continuing.

Introduction
------------

A common topology for Charmed OpenStack is the co-location of the nova-compute
and the ceph-osd applications. This article covers the removal and redeployment
of such a data plane cloud node.

.. important::

   For the target cloud node, only nova-compute, ceph-osd, and their
   subordinate applications are assumed to be deployed.

.. warning::

   Migration involves disabling compute services on the source host,
   effectively removing the hypervisor from the cloud.

Procedure
---------

First ensure that the cloud is in a healthy state, the Compute services and
Ceph cluster in particular.

Identify cloud node specifics
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Identify the unit and hypervisor name of the compute node:

.. code-block:: none

   juju status nova-compute
   openstack hypervisor list

Identify the unit of the storage node and the IDs of the associated OSDs:

.. code-block:: none

   juju status ceph-osd
   juju exec -a ceph-osd mount | grep ceph
   juju ssh ceph-mon/leader sudo ceph osd tree

In this example,

* the existing compute node:

  * is hosted on unit ``nova-compute/0``
  * has a hypervisor name of node2.maas

* the existing storage node:

  * is hosted on unit ``ceph-osd/2``
  * has two OSDs present and their IDs are 0 and 1

The ID of the existing Juju machine is common to both applications and is
assumed to be 14.

The ID of the replacement Juju machine is assumed to be 21.

* the new storage node:

  * will be hosted on unit ``ceph-osd/10``
  * will have storage disks ``/dev/nvme0n1 /dev/nvme0n2``

.. warning::

   You must replace the values in this example with the values of your actual
   environment.

Disable nova-compute services
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Disable nova-compute services on the node:

.. code-block:: none

   juju run --wait nova-compute/0 disable

Respawn any Octavia VMs
~~~~~~~~~~~~~~~~~~~~~~~

Skip this section if Octavia is not deployed in the cloud.

Any possible Octavia load balancer VMs (amphorae) need to be identified and
respawned.

.. note::

   Migrating the amphorae like any other VMs (see next section) may work but
   the Octavia project recommends respawning (failing over) its VMs. This is
   because migration may take longer than expected, which may in turn cause
   Octavia to see its VMs as lost/stale. See `Evacuating a Specific Amphora
   from a Host`_ in the upstream documentation.

List the amphorae hosted on the node:

.. code-block:: none

   openstack server list --host node2.maas --all-projects | grep amphora

The Amphora ID is appended to the VM name.

For each VM,

#. gather the load balancer ID:

   .. code-block:: none

      openstack loadbalancer amphora show <Amphora ID>

#. respawn an Octavia VM and monitor its progress:

   .. code-block:: none

      openstack loadbalancer failover <LB ID>
      watch 'openstack loadbalancer amphora list | grep <LB ID>'

   The original VM will be removed from the compute node.

Live migrate the compute node VMs
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Evacuate the compute node's VMs by live migration:

.. code-block:: none

   nova host-evacuate-live node2.maas

See cloud operation :doc:`Live migrate VMs from a running compute node
<ops-live-migrate-vms>` for in-depth coverage of live migration.

Ensure that all VMs have been evacuated:

.. code-block:: none

   juju ssh nova-compute/0 sudo virsh list --all

Unregister objects from the cloud
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Unregister the compute node
^^^^^^^^^^^^^^^^^^^^^^^^^^^

Unregister the compute node from the cloud:

.. code-block:: none

   juju run --wait nova-compute/0 remove-from-cloud

See cloud operation :ref:`Scale back the nova-compute application
<unregister_compute_node>` for more details on this step.

Unregister the neutron agents
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Unregister the associated neutron agent from the cloud. The agent's ID should
be the compute node's name. Verify this by first listing the agents:

.. code-block:: none

   openstack network agent list
   openstack network agent delete node2.maas

Remove OSD storage devices
~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: none

   juju run --wait ceph-osd/2 remove-disk osd-ids=osd.0 purge=true
   juju run --wait ceph-osd/2 remove-disk osd-ids=osd.1 purge=true

.. note::

   The Ceph operation `Removing OSDs`_ has more details on the ``remove-disk``
   action.

Remove and add a Juju machine
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Remove the affected Juju machine from the model:

.. code-block:: none

   juju remove-machine 14

Add a Juju machine

.. code-block:: none

   juju add-machine

The machine's hardware requirements can be stated via the ``--constraints``
option. This option can also be used to select a particular MAAS node by
specifying a MAAS tag. The chosen machine should have the storage devices
necessary to compensate for the Ceph OSDs that were removed.

Add Ceph storage and compute services
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Add Ceph storage and compute services to the new Juju machine:

.. code-block:: none

   juju add-unit nova-compute --to 21
   juju add-unit ceph-osd --to 21

Integrate the new Ceph disks
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The current value of the ceph-osd charm option ``osd-devices`` may match the
two storage devices belonging to the new cloud node. In such a case, there is
nothing else to do; the disks will be integrated into the cluster
automatically.

First list all the disks on the new storage node:

.. code-block:: none

   juju run --wait ceph-osd/10 list-disks

Then query the charm option:

.. code-block:: none

   juju config ceph-osd osd-devices

If the new disk is not represented by the option's value you can either change
the value (which applies to the entire cluster) or use the `add-disk` action
against the new ceph-osd unit. Here, we'll use the action using our
previously-assumed values:

.. code-block:: none

   juju run --wait ceph-osd/10 add-disk \
      osd-devices='/dev/nvme0n1 /dev/nvme0n2'

Inspect Ceph cluster changes
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

It is recommended to get a summary of the Ceph cluster using the commands used
previously. In particular, the ceph-osd unit number will have changed:

.. code-block:: none

   juju status ceph-osd
   juju exec -a ceph-osd mount | grep ceph
   juju ssh ceph-mon/leader sudo ceph osd tree

Customise the local environment
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Perform any customisations that may be required as per the local environment.
This may include:

#. Adding the new compute node to a Nova aggregate or availability zone
#. Setting CRUSH device classes for the new Ceph OSDs

Verify the new cloud node
~~~~~~~~~~~~~~~~~~~~~~~~~

The hyperconverged Ceph storage and compute node has now been replaced.

Verify that the new compute node is functional. See the verification step in
cloud operation `Scale out the nova-compute application
<scale_out_nova_compute_verfication>` for guidance.

Verify that the Ceph cluster is healthy:

.. code-block:: none

   juju ssh ceph-mon/leader sudo ceph status

.. LINKS
.. _nova-compute charm: https://charmhub.io/nova-compute
.. _Evacuating a Specific Amphora from a Host: https://docs.openstack.org/octavia/latest/admin/guides/operator-maintenance.html#evacuating-a-specific-amphora-from-a-host
.. _Removing OSDs: https://ubuntu.com/ceph/docs/removing-osds
