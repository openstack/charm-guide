:orphan:

============================================
Live migrate VMs from a running compute node
============================================

.. important::

   This page has been identified as being affected by the breaking changes
   introduced between versions 2.9.x and 3.x of the Juju client. Read
   support note :ref:`juju_29_3x_changes` before continuing.

Introduction
------------

A VM migration is the relocation of a VM from one hypervisor to another.

When a VM has a live migration performed it is not shut down during the
process. This is useful when there is an imperative to not interrupt the
applications that are running on the VM.

This article covers manual migrations (migrating an individual VM) and node
evacuations (migrating all VMs on a compute node).

.. warning::

   * Migration involves disabling compute services on the source host,
     effectively removing the hypervisor from the cloud.

   * Network usage may be significantly impacted if block migration mode is
     used.

   * VMs with intensive memory workloads may require pausing for live
     migration to succeed.

Terminology
-----------

This article makes use of the following terms:

block migration
  A migration mode where a disk is copied over the network (source host to
  destination host). It works with local storage only.

boot-from-volume
  A root disk that is based on a Cinder volume.

ephemeral disk
  A non-root disk that is managed by Nova. It is based on local or shared
  storage.

local storage
  Storage that is local to the hypervisor. Disk images are typically found
  under ``/var/lib/nova/instances``.

nova-scheduler
  The cloud's ``nova-scheduler`` can be leveraged in the context of migrations
  to select destination hosts dynamically. It is compatible with both manual
  migrations and node evacuations.

shared storage
  Storage that is accessible to multiple hypervisors simultaneously (e.g. Ceph
  RBD, NFS, iSCSI).

Procedure
---------

Ensure live migration is enabled
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

By default, live migration is disabled. Check the current configuration:

.. code-block:: none

   juju config nova-compute enable-live-migration

Enable the functionality if the command returns 'false':

.. code-block:: none

   juju config nova-compute enable-live-migration=true

See the `nova-compute charm`_ for information on all migration-related
configuration options. Also see section `SSH keys and VM migration`_ for
information on how multiple application groups can affect migrations.

Gather relevant information
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Display the nova-compute units:

.. code-block:: none

   juju status nova-compute

This article will be based on the command's following partial output:

.. code-block:: console

   Unit              Workload  Agent  Machine  Public address  Ports    Message
   nova-compute/0*   active    idle   0        10.0.0.222               Unit is ready
     ntp/0*          active    idle            10.0.0.222      123/udp  chrony: Ready
     ovn-chassis/0*  active    idle            10.0.0.222               Unit is ready
   nova-compute/1    active    idle   3        10.0.0.241               Unit is ready
     ntp/1           active    idle            10.0.0.241      123/udp  chrony: Ready
     ovn-chassis/1   active    idle            10.0.0.241               Unit is ready

List the compute node hostnames:

.. code-block:: none

   openstack hypervisor list

   +----+---------------------+-----------------+------------+-------+
   | ID | Hypervisor Hostname | Hypervisor Type | Host IP    | State |
   +----+---------------------+-----------------+------------+-------+
   |  1 | node1.maas          | QEMU            | 10.0.0.222 | up    |
   |  2 | node4.maas          | QEMU            | 10.0.0.241 | up    |
   +----+---------------------+-----------------+------------+-------+

Based on the above, map units to node names. This information will be useful
later on. The source host should also be clearly identified (this document will
use 'node1.maas'):

+------------+----------------+-------------+
| Node name  | Unit           | Source host |
+============+================+=============+
| node1.maas | nova-compute/0 | ✔           |
+------------+----------------+-------------+
| node4.maas | nova-compute/1 |             |
+------------+----------------+-------------+

List the VMs hosted on the source host:

.. code-block:: none

   openstack server list --host node1.maas --all-projects

   +--------------------------------------+---------+--------+----------------------------------+-------------+----------+
   | ID                                   | Name    | Status | Networks                         | Image       | Flavor   |
   +--------------------------------------+---------+--------+----------------------------------+-------------+----------+
   | 81df1304-f755-4ae6-9b8c-2f888f6ad623 | focal-2 | ACTIVE | int_net=192.168.0.144, 10.0.0.76 | focal-amd64 | m1.micro |
   | 7e897540-a0aa-4031-9b7c-dd03ebc8ec5e | focal-3 | ACTIVE | int_net=192.168.0.73, 10.0.0.69  | focal-amd64 | m1.micro |
   +--------------------------------------+---------+--------+----------------------------------+-------------+----------+

Ensure adequate capacity on the destination host
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Oversubscribing the destination host can lead to service outages. This is an
issue when a destination host is explicitly selected by the operator.

The following commands are useful for discovering a VM's flavor, listing flavor
parameters, and viewing the available capacity of a destination host:

.. code-block:: none

   openstack server show <vm-name> -c flavor
   openstack flavor show <flavor> -c vcpus -c ram -c disk
   openstack host show <destination-host>

Avoid expired Keystone tokens
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Keystone token expiration times should be increased when dealing with oversized
VMs as expired tokens will prevent the cloud database from being updated. This
will lead to migration failure and a corrupted database entry (see
:ref:`keystone_tokens`).

To set the token expiration time to three hours (from the default one hour):

.. code-block:: none

   juju config keystone token-expiration=10800

To ensure that the new expiration time is in effect, wait for the current
tokens to expire (e.g. one hour) before continuing.

Disable the source host
~~~~~~~~~~~~~~~~~~~~~~~

Prior to migration or evacuation, disable the source host by referring to its
corresponding unit:

.. code-block:: none

   juju run nova-compute/0 disable

This will stop nova-compute services and inform nova-scheduler to no longer
assign new VMs to the host.

Live migrate VMs
~~~~~~~~~~~~~~~~

Live migrate VMs using either manual migration or node evacuation.

Manual migration
^^^^^^^^^^^^^^^^

The command to use when live migrating VMs manually is:

.. code-block:: none

   openstack server migrate --live-migration [--block-migration] [--host <dest-host>] <vm>

Examples are provided for various scenarios.

.. note::

   * Depending on your client the Nova API Microversion of '2.30' may need to
     be specified when combining live migration with a specified host (i.e.
     ``--os-compute-api-version 2.30``).

   * Specifying a destination host will override any anti-affinity rules that
     may be in place.

.. important::

   The migration of VMs using local storage will fail if the
   ``--block-migration`` option is not specified. However, the use of this
   option will also lead to a successful migration for a combination of local
   and non-local storage (e.g. local ephemeral disk and boot-from-volume).

1. To migrate VM 'focal-2', which is backed by local storage, using the
   scheduler:

   .. code-block:: none

      openstack server migrate --live-migration --block-migration focal-2

2. To migrate VM 'focal-3', which is backed by non-local storage, using the
   scheduler:

   .. code-block:: none

      openstack server migrate --live-migration focal-3

3. To migrate VM 'focal-2', which is backed by a combination local and
   non-local storage:

   .. code-block:: none

      openstack server migrate --live-migration --block-migration focal-2

4. To migrate VM 'focal-2', which is backed by local storage, to host
   'node4.maas' specifically:

   .. code-block:: none

      openstack --os-compute-api-version 2.30 server migrate \
         --live-migration --block-migration --host node4.maas focal-2

5. To migrate VM 'focal-3', which is backed by non-local storage, to host
   'node4.maas' specifically:

   .. code-block:: none

      openstack --os-compute-api-version 2.30 server migrate \
         --live-migration --host node4.maas focal-3

Node evacuation
^^^^^^^^^^^^^^^

The command to use when live evacuating a compute node is:

.. code-block:: none

   nova host-evacuate-live [--block-migrate] [--target-host <dest-host>] <source-host>

Examples are provided for various scenarios.

.. note::

   * The scheduler may send VMs to multiple destination hosts.

   * Block migration will be attempted on VMs by default.

.. important::

   The migration of VMs using non-local storage will fail if the
   ``--block-migrate`` option is specified. However, the omittance of this
   option will lead to a successful migration for a combination of local and
   non-local storage (e.g. local ephemeral disk and boot-from-volume). This
   option therefore has no compelling use case.

1. To evacuate host 'node1.maas' of VMs, which are backed by local or non-local
   storage (or a combination thereof):

.. code-block:: none

   nova host-evacuate-live node1.maas

2. To evacuate host 'node1.maas' of VMs, which are backed by local or non-local
   storage (or a combination thereof), to host 'node4.maas' specifically:

.. code-block:: none

   nova host-evacuate-live --target-host node4.maas node1.maas

Enable the source host
~~~~~~~~~~~~~~~~~~~~~~

Providing the source host is not being retired, re-enable it by referring to
its corresponding unit:

.. code-block:: none

   juju run nova-compute/0 enable

This will start nova-compute services and allows nova-scheduler to run new VMs
on this host.

Revert Keystone token expiration time
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If the Keystone token expiration time was modified in an earlier step, change
it back to its original value. Here it is reset to the default of one hour:

.. code-block:: none

   juju config keystone token-expiration=3600

Troubleshooting
---------------

Migration list and migration ID
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To get a record of all past migrations on a per-VM basis, which includes the
migration ID:

.. code-block:: none

   nova migration-list

In this output columns have been removed for legibility purposes, and only a
single, currently running, migration is shown:

.. code-block:: console

   +----+-------------+------------+---------+--------------------------------------+----------------+
   | Id | Source Node | Dest Node  | Status  | Instance UUID                        | Type           |
   +----+-------------+------------+---------+--------------------------------------+----------------+
   | 29 | node4.maas  | node1.maas | running | 81df1304-f755-4ae6-9b8c-2f888f6ad623 | live-migration |
   +----+-------------+------------+---------+--------------------------------------+----------------+

The above migration has an ID of '29'.

Forcing a migration
~~~~~~~~~~~~~~~~~~~

A VM with an intensive memory workload can be hard to live migrate. In such a
case the migration can be forced by pausing the VM until the copying of memory
is finished.

The :command:`openstack server show` command output contains a ``progress``
field that normally displays a value of '0' but for this scenario of a busy VM
it will start to provide percentages.

Forcing a migration should be considered once its progress nears 90%:

.. code-block:: none

   nova live-migration-force-complete <vm> <migration-id>

.. caution::

   Some applications are time sensitive and may not tolerate a forced migration
   due to the effect pausing can have on a VM's clock.

Aborting a migration
~~~~~~~~~~~~~~~~~~~~

A migration can be aborted like this:

.. code-block:: none

   nova live-migration-abort <vm> <migration-id>

Migration logs
~~~~~~~~~~~~~~

A failed migration will result in log messages being appended to the
``nova-compute.log`` file on the source host.

.. LINKS
.. _nova-compute charm: https://charmhub.io/nova-compute
.. _SSH keys and VM migration: https://opendev.org/openstack/charm-nova-compute/src/branch/master/README.md#ssh-keys-and-vm-migration
