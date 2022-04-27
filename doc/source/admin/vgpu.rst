==================
Virtual GPU (vGPU)
==================

Overview
--------

Virtual GPU (vGPU) is a graphics virtualisation solution that provides virtual
machines simultaneous access to physical GPUs hosted on hypervisors. The
physical GPU must be expressly designed for this purpose.

This page explains how to add vGPU capability to an existing cloud.

.. note::

   Due to several upstream OpenStack limitations, the vGPU feature generally
   works better on OpenStack Stein (and newer).

.. warning::

   The enablement of the vGPU feature will require a reboot of all affected
   hypervisors.

Pre-requisites
--------------

The following requirements must be met in order to use the vGPU feature:

* OpenStack Queens or newer
* `Nvidia vGPU software`_ (Linux driver ``510.47.03`` or newer; Ubuntu package)
* a `supported GPU device on Ubuntu`_

.. note::

   The OpenStack charms do not have multiple vGPU support (multiple vGPUs
   assigned to a single VM).

Deployment
----------

Deploy the nova-compute-nvidia-vgpu application and add a relation to
nova-compute:

.. code-block:: none

   juju deploy ch:nova-compute-nvidia-vgpu --channel=yoga/stable
   juju add-relation nova-compute-nvidia-vgpu:nova-vgpu nova-compute:nova-vgpu

.. tip::

   If the vGPU feature is to be used to a significant extent it is recommended
   to leverage application groups (custom application names) when deploying the
   nova-compute-nvidia-vgpu charm. Doing so will make working with vGPU more
   versatile.

Attach the device drivers to the nova-compute-nvidia-vgpu charm:

.. code-block:: none

   juju attach nova-compute-nvidia-vgpu \
      nvidia-vgpu-software=./nvidia-vgpu-ubuntu-510_510.47.03_amd64.deb

Once the model settles, the :command:`juju status nova-compute` command will
show output similar to:

.. code-block:: console

   App                       Version  Status   Scale  Charm                     Channel      Rev  Exposed  Message
   nova-compute              25.0.0   active       1  nova-compute              yoga/edge    579  no       Unit is ready
   nova-compute-nvidia-vgpu           blocked      1  nova-compute-nvidia-vgpu  yoga/edge     13  no       NVIDIA GPU found; installed NVIDIA software: 510.47.03; reboot required?
   ovn-chassis               22.03.0  active       1  ovn-chassis               22.03/edge    45  no       Unit is ready

   Unit                           Workload  Agent  Machine  Public address  Ports  Message
   nova-compute/0*                active    idle   1        10.246.114.59          Unit is ready
     nova-compute-nvidia-vgpu/0*  blocked   idle            10.246.114.59          NVIDIA GPU found; installed NVIDIA software: 510.47.03; reboot required?
     ovn-chassis/0*               active    idle            10.246.114.59          Unit is ready

When the cloud is ready, reboot all affected Compute nodes:

.. code-block:: none

   juju run -a nova-compute-nvidia-vgpu -- sudo reboot

vGPU type definition
--------------------

One or more physical GPUs can be installed on a Compute node. Each of these
physical GPUs can be divided into one or more vGPUs types, where a type can
support a specific number of actual vGPUs.

A Compute node's vGPU types (and associated physical GPUs) can be listed by
querying the corresponding nova-compute-nvidia-vgpu application unit:

.. code-block:: none

   juju run-action --wait nova-compute-nvidia-vgpu/0 list-vgpu-types

Sample output:

.. code-block:: console

   nvidia-256, 0000:41:00.0, GRID RTX6000-1Q, num_heads=4, frl_config=60, framebuffer=1024M, max_resolution=5120x2880, max_instance=24
   nvidia-257, 0000:41:00.0, GRID RTX6000-2Q, num_heads=4, frl_config=60, framebuffer=2048M, max_resolution=7680x4320, max_instance=12
   nvidia-258, 0000:41:00.0, GRID RTX6000-3Q, num_heads=4, frl_config=60, framebuffer=3072M, max_resolution=7680x4320, max_instance=8
   nvidia-259, 0000:41:00.0, GRID RTX6000-4Q, num_heads=4, frl_config=60, framebuffer=4096M, max_resolution=7680x4320, max_instance=6
   nvidia-260, 0000:41:00.0, GRID RTX6000-6Q, num_heads=4, frl_config=60, framebuffer=6144M, max_resolution=7680x4320, max_instance=4
   nvidia-261, 0000:41:00.0, GRID RTX6000-8Q, num_heads=4, frl_config=60, framebuffer=8192M, max_resolution=7680x4320, max_instance=3
   nvidia-262, 0000:41:00.0, GRID RTX6000-12Q, num_heads=4, frl_config=60, framebuffer=12288M, max_resolution=7680x4320, max_instance=2
   nvidia-263, 0000:41:00.0, GRID RTX6000-24Q, num_heads=4, frl_config=60, framebuffer=24576M, max_resolution=7680x4320, max_instance=1
   nvidia-343, 0000:41:00.0, GRID RTX6000-4C, num_heads=1, frl_config=60, framebuffer=4096M, max_resolution=4096x2160, max_instance=6
   nvidia-344, 0000:41:00.0, GRID RTX6000-6C, num_heads=1, frl_config=60, framebuffer=6144M, max_resolution=4096x2160, max_instance=4
   nvidia-345, 0000:41:00.0, GRID RTX6000-8C, num_heads=1, frl_config=60, framebuffer=8192M, max_resolution=4096x2160, max_instance=3
   nvidia-346, 0000:41:00.0, GRID RTX6000-12C, num_heads=1, frl_config=60, framebuffer=12288M, max_resolution=4096x2160, max_instance=2
   nvidia-347, 0000:41:00.0, GRID RTX6000-24C, num_heads=1, frl_config=60, framebuffer=24576M, max_resolution=4096x2160, max_instance=1
   nvidia-435, 0000:41:00.0, GRID RTX6000-1B, num_heads=4, frl_config=45, framebuffer=1024M, max_resolution=5120x2880, max_instance=24
   nvidia-436, 0000:41:00.0, GRID RTX6000-2B, num_heads=4, frl_config=45, framebuffer=2048M, max_resolution=5120x2880, max_instance=12
   nvidia-437, 0000:41:00.0, GRID RTX6000-1A, num_heads=1, frl_config=60, framebuffer=1024M, max_resolution=1280x1024, max_instance=24
   nvidia-438, 0000:41:00.0, GRID RTX6000-2A, num_heads=1, frl_config=60, framebuffer=2048M, max_resolution=1280x1024, max_instance=12
   nvidia-439, 0000:41:00.0, GRID RTX6000-3A, num_heads=1, frl_config=60, framebuffer=3072M, max_resolution=1280x1024, max_instance=8
   nvidia-440, 0000:41:00.0, GRID RTX6000-4A, num_heads=1, frl_config=60, framebuffer=4096M, max_resolution=1280x1024, max_instance=6
   nvidia-441, 0000:41:00.0, GRID RTX6000-6A, num_heads=1, frl_config=60, framebuffer=6144M, max_resolution=1280x1024, max_instance=4
   nvidia-442, 0000:41:00.0, GRID RTX6000-8A, num_heads=1, frl_config=60, framebuffer=8192M, max_resolution=1280x1024, max_instance=3
   nvidia-443, 0000:41:00.0, GRID RTX6000-12A, num_heads=1, frl_config=60, framebuffer=12288M, max_resolution=1280x1024, max_instance=2
   nvidia-444, 0000:41:00.0, GRID RTX6000-24A, num_heads=1, frl_config=60, framebuffer=24576M, max_resolution=1280x1024, max_instance=1
   nvidia-105, 0000:c1:00.0, GRID V100-1Q, num_heads=4, frl_config=60, framebuffer=1024M, max_resolution=5120x2880, max_instance=16
   nvidia-106, 0000:c1:00.0, GRID V100-2Q, num_heads=4, frl_config=60, framebuffer=2048M, max_resolution=7680x4320, max_instance=8
   nvidia-107, 0000:c1:00.0, GRID V100-4Q, num_heads=4, frl_config=60, framebuffer=4096M, max_resolution=7680x4320, max_instance=4
   nvidia-108, 0000:c1:00.0, GRID V100-8Q, num_heads=4, frl_config=60, framebuffer=8192M, max_resolution=7680x4320, max_instance=2
   nvidia-109, 0000:c1:00.0, GRID V100-16Q, num_heads=4, frl_config=60, framebuffer=16384M, max_resolution=7680x4320, max_instance=1
   nvidia-110, 0000:c1:00.0, GRID V100-1A, num_heads=1, frl_config=60, framebuffer=1024M, max_resolution=1280x1024, max_instance=16
   nvidia-111, 0000:c1:00.0, GRID V100-2A, num_heads=1, frl_config=60, framebuffer=2048M, max_resolution=1280x1024, max_instance=8
   nvidia-112, 0000:c1:00.0, GRID V100-4A, num_heads=1, frl_config=60, framebuffer=4096M, max_resolution=1280x1024, max_instance=4
   nvidia-113, 0000:c1:00.0, GRID V100-8A, num_heads=1, frl_config=60, framebuffer=8192M, max_resolution=1280x1024, max_instance=2
   nvidia-114, 0000:c1:00.0, GRID V100-16A, num_heads=1, frl_config=60, framebuffer=16384M, max_resolution=1280x1024, max_instance=1
   nvidia-115, 0000:c1:00.0, GRID V100-1B, num_heads=4, frl_config=45, framebuffer=1024M, max_resolution=5120x2880, max_instance=16
   nvidia-163, 0000:c1:00.0, GRID V100-2B, num_heads=4, frl_config=45, framebuffer=2048M, max_resolution=5120x2880, max_instance=8
   nvidia-217, 0000:c1:00.0, GRID V100-2B4, num_heads=4, frl_config=45, framebuffer=2048M, max_resolution=5120x2880, max_instance=8
   nvidia-247, 0000:c1:00.0, GRID V100-1B4, num_heads=4, frl_config=45, framebuffer=1024M, max_resolution=5120x2880, max_instance=16
   nvidia-299, 0000:c1:00.0, GRID V100-4C, num_heads=1, frl_config=60, framebuffer=4096M, max_resolution=4096x2160, max_instance=4
   nvidia-300, 0000:c1:00.0, GRID V100-8C, num_heads=1, frl_config=60, framebuffer=8192M, max_resolution=4096x2160, max_instance=2
   nvidia-301, 0000:c1:00.0, GRID V100-16C, num_heads=1, frl_config=60, framebuffer=16384M, max_resolution=4096x2160, max_instance=1

Here, 40 vGPU types are available from among two physical GPU devices:

* ``0000:41:00.0``
* ``0000:c1:00.0``

The last column of each type's entry gives the number of vGPU cards that can be
assigned to cloud VMs (e.g. ``max_instance=4``).

vGPU type selection
-------------------

vGPUs are made available to the cloud based on the selection of one or more
vGPU types.

The selection should be based on the knowledge of all types across the cloud.
The types for each Compute node should therefore first be listed before making
a decision.

Selecting a vGPU type consists of mapping it to a physical GPU device(s).
Multiple types can also be selected but note that a physical GPU can only be
associated with one type. See the Nova documentation (`Attaching virtual GPU
devices to guests`_) for upstream information.

.. important::

   On OpenStack releases older than Stein, only one vGPU type can be selected.

The simplest case is a mapping of one vGPU type to a single physical GPU. For
example, to have three vGPUs become available (``max_instance=3``), vGPU type
``nvidia-442`` (on physical GPU ``0000:41:00.0``) can be selected:

.. code-block:: none

   juju config nova-compute-nvidia-vgpu vgpu-device-mappings="{'nvidia-442': ['0000:41:00.0']}"

.. warning::

   Changing vGPU types may prevent new VMs from being created. Failure will
   occur if a new VM uses a type that sollicites the same physical GPU of any
   existing VM. Recall that a physical GPU can only support one vGPU type at
   any given time. This can be mitigated through the strategic use of
   appilcation groups for nova-compute and/or nova-compute-nvidia-vgpu.

Once the model has settled, the vGPUs can be listed via the OpenStack CLI.
Start by listing the physical GPUs:

.. code-block:: none

   openstack resource provider list
   +--------------------------------------+-----------------------------------+------------+--------------------------------------+--------------------------------------+
   | uuid                                 | name                              | generation | root_provider_uuid                   | parent_provider_uuid                 |
   +--------------------------------------+-----------------------------------+------------+--------------------------------------+--------------------------------------+
   | e0f99e40-a7a5-42bb-a222-387a540c3725 | node-sparky.maas                  |          3 | e0f99e40-a7a5-42bb-a222-387a540c3725 | None                                 |
   | 807d28f4-4b30-4f85-a770-1bcebd1236d3 | node-sparky.maas_pci_0000_41_00_0 |          1 | e0f99e40-a7a5-42bb-a222-387a540c3725 | e0f99e40-a7a5-42bb-a222-387a540c3725 |
   +--------------------------------------+-----------------------------------+------------+--------------------------------------+--------------------------------------+

.. important::

   Only starting with OpenStack Stein are physical GPU devices (second row)
   abstracted from their root provider (first row).

Here, there is a single physical GPU with an OpenStack UUID of of
``807d28f4-4b30-4f85-a770-1bcebd1236d3``.

.. note::

   To get the last two columns above (not necessary), the
   :command:`openstackclients` snap must at least be at version
   ``xena/stable``.

A physical GPU, on Stein or newer, can now be queried via its UUID:

.. code-block:: none

   openstack resource provider inventory list 807d28f4-4b30-4f85-a770-1bcebd1236d3
   +----------------+------------------+----------+----------+----------+-----------+-------+------+
   | resource_class | allocation_ratio | min_unit | max_unit | reserved | step_size | total | used |
   +----------------+------------------+----------+----------+----------+-----------+-------+------+
   | VGPU           |              1.0 |        1 |        3 |        0 |         1 |     3 |    0 |
   +----------------+------------------+----------+----------+----------+-----------+-------+------+

There is a total of three vGPUs available.

OpenStack configuration
-----------------------

vGPUs are assigned to VMs by means of an OpenStack flavor.

The following example configures an existing flavor to use one vGPU (optionally
create a new flavor):

.. code-block:: none

   openstack flavor set <flavor-name> \
     --property resources:VGPU=1

Upon creation of a VM with such a flavor the number of used vGPUs will increase
by one. This can be verified by a new physical GPU query:

.. code-block:: none

   openstack resource provider inventory list 807d28f4-4b30-4f85-a770-1bcebd1236d3
   +----------------+------------------+----------+----------+----------+-----------+-------+------+
   | resource_class | allocation_ratio | min_unit | max_unit | reserved | step_size | total | used |
   +----------------+------------------+----------+----------+----------+-----------+-------+------+
   | VGPU           |              1.0 |        1 |        3 |        0 |         1 |     3 |    1 |
   +----------------+------------------+----------+----------+----------+-----------+-------+------+

Other query methods
~~~~~~~~~~~~~~~~~~~

An individual VM can be queried for vGPU information:

.. code-block:: none

   openstack resource provider allocation show <vm-uuid>

On the associated hypervisor, at the libvirt level, the XML definition of the
VM (:command:`virsh dumpxml <domain>`) will contain a ``hostdev`` stanza that
represents the vGPU card:

.. code-block:: console

   <hostdev mode='subsystem' type='mdev' managed='no' model='vfio-pci' display='off'>
     <source>
       <address uuid='b2107403-110c-45b0-af87-32cc91597b8a'/>
     </source>
     <alias name='hostdev0'/>
     <address type='pci' domain='0x0000' bus='0x00' slot='0x04' function='0x0'/>
   </hostdev>

On the VM itself the card can be exposed via the :command:`lspci` command:

.. code-block:: console

   00:04.0 VGA compatible controller: NVIDIA Corporation TU102GL [Quadro RTX 6000/8000] (rev a1) (prog-if 00 [VGA controller])
           Subsystem: NVIDIA Corporation TU102GL [Quadro RTX 6000/8000]
           Physical Slot: 4
           Flags: fast devsel, IRQ 11
           Memory at fc000000 (32-bit, non-prefetchable) [size=16M]
           Memory at e0000000 (64-bit, prefetchable) [size=256M]
           Memory at fa000000 (64-bit, non-prefetchable) [size=32M]
           Expansion ROM at 000c0000 [virtual] [disabled] [size=128K]
           Capabilities: [d0] Vendor Specific Information: Len=1b <?>
           Capabilities: [68] MSI: Enable- Count=1/1 Maskable- 64bit+
           Kernel modules: nvidiafb

Targeting specific vGPU types
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

For Compute nodes that are associated with multiple vGPU types it may be useful
to state what type a VM should use. This is ultimately specified via a physical
GPU since the latter is always mapped to a single vGPU type. It is achieved by
means of a Placement trait.

.. note::

   A trait is a hardware constraint that is used by the cloud's scheduler (see
   upstream documentation: `Placement API`_).

Do this by creating a trait and allocating it to a physical GPU:

.. code-block:: none

   openstack --os-placement-api-version 1.6 trait create CUSTOM_NVIDIA_442
   openstack --os-placement-api-version 1.6 resource provider trait set --trait CUSTOM_NVIDIA_442 807d28f4-4b30-4f85-a770-1bcebd1236d3

.. important::

   On releases older than Stein, since the UUID of a physical GPU is not
   available, a trait cannot be created.

The following example configures an existing flavor to require the
'CUSTOM_NVIDIA_442' trait (optionally create a new flavor):

.. code-block:: none

   openstack flavor set <flavor-name> \
     --property resources:VGPU=1 \
     --property trait:CUSTOM_NVIDIA_442=required

.. LINKS
.. _Nvidia vGPU software: https://docs.nvidia.com/grid/index.html
.. _supported GPU device on Ubuntu: https://docs.nvidia.com/grid/14.0/grid-vgpu-release-notes-ubuntu/index.html#hardware-configuration
.. _Attaching virtual GPU devices to guests: https://docs.openstack.org/nova/latest/admin/virtual-gpu.html
.. _Placement API: https://docs.openstack.org/api-ref/placement/#traits
