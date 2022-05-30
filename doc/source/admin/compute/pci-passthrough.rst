===============
PCI Passthrough
===============

Introduction
++++++++++++

PCI pass through allows compute nodes to pass a physical PCI device to a hosted
VM. This can be used for direct access to a PCI device inside the VM. For
example a GPU or direct access to a physical network interface. The OpenStack
charms fully support this feature. The following will document deployment and
configuration of the feature. This document will focus on using OpenStack
charms, Juju and MAAS. It is worth familiarizing yourself with the generic
OpenStack Nova documentation at
https://docs.openstack.org/nova/latest/admin/pci-passthrough.html.


Example Hardware
~~~~~~~~~~~~~~~~

This document will assume a MAAS environment with compute hosts that have two
GPUs per physical host. In this case, a Quadro P5000 with PCI IDs **83:00.0**
and **84:00.0** and vendor and product ids of **10de:1bb0**.

lspci output:

.. code:: bash

  lspci -nn | grep VGA
  83:00.0 VGA compatible controller [0300]: NVIDIA Corporation Device [10de:1bb0] (rev a1) (prog-if 00 [VGA controller])
  84:00.0 VGA compatible controller [0300]: NVIDIA Corporation Device [10de:1bb0] (rev a1) (prog-if 00 [VGA controller])


Infrastructure
++++++++++++++

.. note::
 To passthrough PCI devices IOMMU must be enabled for the hardware. Depending
 on the hardware vendor (Intel or AMD) enable the virtualisation feature in
 BIOS and set the correct kernel parameter as described bellow (intel_iommu,
 amd_iommu).




Blacklisting the PCI device
~~~~~~~~~~~~~~~~~~~~~~~~~~~

There are certain cases, particularly for GPUs, where we must blacklist the PCI
device at the physical host so that its kernel does not load drivers. i.e. stop
the physical host's kernel from loading NVIDIA or nouveau drivers for the GPU.
The host must be able to cleanly pass the device to libvirt.


Most PCI devices can be hot unbound and then bound via vfio-pci or the pci-stub
legacy method and therefore do not require blacklisting. However, GPU devices
do require blacklisting. This documentation covers the GPU case. The
blacklisting documentation is based on the following:
https://www.pugetsystems.com/labs/articles/Multiheaded-NVIDIA-Gaming-using-Ubuntu-14-04-KVM-585/


MAAS
~~~~

MAAS tags can also send kernel boot parameters. The tag will therefore serve
two purposes: to select physical hosts which have PCI devices installed and to
pass kernel parameters in order to blacklist the PCI device stopping the kernel
from loading a driver for the device.

.. code:: bash

 maas $MASS_PROFILE tags create name="pci-pass" comment="PCI Passthrough kernel parameters" kernel_opts="intel_iommu=on vfio_iommu_type1.allow_unsafe_interrupts=1"

.. note::
 A MAAS host with more than one tag that sets kernel parameters will execute only one set. Good practice is to only have one kernel parameter setting tag on any given host to avoid confusion.

In Target Changes
~~~~~~~~~~~~~~~~~

The MAAS tag and kernel parameters are only half the blacklisting solution. We
also need the host to actively check the blacklisting. This requires setting up
some configuration at deploy time. There are a few options on how to accomplish
this, however, this document will use cloud init configuration at the Juju
model level.

For other options to customize the deployed host see:
https://docs.maas.io/2.5/en/nodes-custom

Juju can customize the host configuration using model level configuration for
cloudinit-userdata. We will leverage this feature to do the final configuration
of the physical host.

.. note::

 Model level cloudinit userdata will get executed on all "machines" in a model.
 This includes physical hosts and containers. Take care to ensure any
 customization scripts will work in all cases.

Determine the vendor and product ID of the PCI device from lspci. In our case
the GPU's is **10de:1bb0**.

Determine the PCI device IDs of the PCI device from lspci. In our case the
GPU's are **83:00.0** and **84:00.0**.

Determine which driver we are blacklisting. This information can be found in
the lspci output "Kernel modules." In our case **nvidiafb** and **nouveau**.

Base64 encode an /etc/default/grub with the default line set:

.. code:: bash

 echo 'GRUB_DEFAULT=0
 GRUB_HIDDEN_TIMEOUT=0
 GRUB_HIDDEN_TIMEOUT_QUIET=true
 GRUB_TIMEOUT=10
 GRUB_DISTRIBUTOR=`lsb_release -i -s 2> /dev/null || echo Debian`
 GRUB_CMDLINE_LINUX_DEFAULT="intel_iommu=on vfio_iommu_type1.allow_unsafe_interrupts=1 modprobe.blacklist=nvidiafb,nouveau"
 GRUB_CMDLINE_LINUX=""' | base64

Base64 encode the following /etc/rc.local script with the correct PCI device
IDS. In our case **0000:83:00.0** and **0000:84:00.0**.

.. code:: bash

 echo 'vfiobind() {
     dev="$1"
     vendor=$(cat /sys/bus/pci/devices/$dev/vendor)
     device=$(cat /sys/bus/pci/devices/$dev/device)
     if [ -e /sys/bus/pci/devices/$dev/driver ]; then
             echo $dev > /sys/bus/pci/devices/$dev/driver/unbind
     fi
     echo $vendor $device > /sys/bus/pci/drivers/vfio-pci/new_id
 }

 vfiobind 0000:83:00.0
 vfiobind 0000:84:00.0' | base64

Take all the information from above to create the following YAML. Set pci_stub,
$BASE64_ENCODED_RC_LOCAL and $BASE64_ENCODED_DEFAULT_GRUB correctly.

.. code:: bash

 cloudinit-userdata: |
   postruncmd:
     - "update-initramfs -u > /root/initramfs-update.log"
     - "update-grub > /root/grub-update.log"
   write_files:
     - path: /etc/initramfs-tools/modules
       content: pci_stub ids=10de:1bb0
     - path: /etc/modules
       content: |
         pci_stub
         vfio
         vfio_iommu_type1
         vfio_pci
     - path: /etc/rc.local
       encoding: b64
       permissions: '0755'
       content: $BASE64_ENCODED_RC_LOCAL
     - path: /etc/default/grub
       encoding: b64
       content: $BASE64_ENCODED_DEFAULT_GRUB

Create the juju model with cloudinit-userdata set with this YAML:

.. code:: bash

 juju add-model openstack-deployment --config cloudinit-userdata.yaml

For further cloud init documentation for customization see:
https://cloudinit.readthedocs.io/en/latest/topics/examples.html


Deploy OpenStack
++++++++++++++++

At this point we are ready to deploy OpenStack using the OpenStack charms with
Juju and MAAS. The charm deployment guide already documents this process. The
only additional settings required are setting the PCI aliases.

Manually:

.. code:: bash

 juju config nova-cloud-controller pci-alias='{"vendor_id":"10de", "product_id":"1bb0", "name":"gpu"}'
 juju config nova-cloud-controller scheduler-default-filters="RetryFilter,AvailabilityZoneFilter,CoreFilter,RamFilter,ComputeFilter,ComputeCapabilitiesFilter,ImagePropertiesFilter,ServerGroupAntiAffinityFilter,ServerGroupAffinityFilter,DifferentHostFilter,SameHostFilter,AggregateInstanceExtraSpecsFilter,PciPassthroughFilter"
 juju config nova-compute pci-alias='{"vendor_id":"10de", "product_id":"1bb0", "name":"gpu"}'
 juju config nova-compute pci-passthrough-whitelist='{ "vendor_id": "10de", "product_id": "1bb0" }'
 # If passing through a GPU use spice for console which creates a usable VGA device for the VMs
 juju config nova-cloud-controller console-access-protocol=spice

Example bundle snippet. Update the OpenStack bundle.

.. code:: bash

 machines:
   '0':
     series: bionic
     # Use the MAAS tag pci-pass for hosts with the PCI device installed.
     constraints: tags=pci-pass
   '1':
     series: bionic
     # Use the inverse (NOT) ^pci-pass tag for hosts without the PCI device.
     constraints: tags=^pci-pass

 applications:
   nova-compute:
     charm: cs:nova-compute
     num_units: 1
     options:
       pci-alias: '{"vendor_id":"10de", "product_id":"1bb0", "name":"gpu"}'
       pci-passthrough-whitelist: '{ "vendor_id": "10de", "product_id": "1bb0" }'
     to:
     - '0'
   nova-cloud-controller:
     charm: cs:nova-cloud-controller
     num_units: 1
     options:
       pci-alias: '{"vendor_id":"10de", "product_id":"1bb0", "name":"gpu"}'
       scheduler-default-filters="RetryFilter,AvailabilityZoneFilter,CoreFilter,RamFilter,ComputeFilter,ComputeCapabilitiesFilter,ImagePropertiesFilter,ServerGroupAntiAffinityFilter,ServerGroupAffinityFilter,DifferentHostFilter,SameHostFilter,AggregateInstanceExtraSpecsFilter,PciPassthroughFilter"
       console-access-protocol: spice
     to:
     - lxd:1

Post Deployment
~~~~~~~~~~~~~~~

Create a flavor. Set the pci_passthrough property with the alias name set
above, in our case **gpu** and the number of devices to pass in this case 1.

.. code:: bash

 openstack flavor create --ram 8192 --disk 100 --vcpu 8 m1.gpu
 openstack flavor set m1.gpu --property "pci_passthrough:alias"="gpu:1"


Boot an instance with the PCI device passed through. Use the flavor just
created:

.. code:: bash

 openstack server create --key-name $KEY --image $IMAGE --nic net-id=$NETWORK --flavor m1.gpu gpu-enabled-vm

SSH onto the VM and run lspci to see the PCI device in the VM. In our case the
NVIDIA **1bb0**.

.. code:: bash

 $ lspci
 00:00.0 Host bridge: Intel Corporation 440FX - 82441FX PMC [Natoma] (rev 02)
 00:01.0 ISA bridge: Intel Corporation 82371SB PIIX3 ISA [Natoma/Triton II]
 00:01.1 IDE interface: Intel Corporation 82371SB PIIX3 IDE [Natoma/Triton II]
 00:01.2 USB controller: Intel Corporation 82371SB PIIX3 USB [Natoma/Triton II] (rev 01)
 00:01.3 Bridge: Intel Corporation 82371AB/EB/MB PIIX4 ACPI (rev 03)
 00:02.0 VGA compatible controller: Red Hat, Inc. QXL paravirtual graphic card (rev 04)
 00:03.0 Ethernet controller: Red Hat, Inc Virtio network device
 00:04.0 Communication controller: Red Hat, Inc Virtio console
 00:05.0 SCSI storage controller: Red Hat, Inc Virtio block device
 00:06.0 VGA compatible controller: NVIDIA Corporation Device 1bb0 (rev a1)
 00:07.0 Unclassified device [00ff]: Red Hat, Inc Virtio memory balloon

Boot an instance without a PCI device passed. Use any flavor without the
pci_passthrough property set. The PciPassthroughFilter will do the right thing.

.. code:: bash

 openstack server create --key-name $KEY --image $IMAGE --nic net-id=$NETWORK --flavor m1.medium no-gpu-vm
