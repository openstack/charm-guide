.. _openstack-on-lxd:

================
OpenStack on LXD
================

Overview
========

A production OpenStack deployment is typically made over a number of physical servers, using LXD containers where appropriate for control plane services.

However, the average developer probably does not have, or want to have, access to such infrastructure for day-to-day charm development.

Its possible to deploy OpenStack using the OpenStack Charms in LXD containers on a single machine; this allows for faster, localized charm development and testing.

The scenarios presented here are for development, testing and demonstration, and should not be used in place of multiple-machine, highly-available OpenStack deployment topologies and configurations more suitable for production use cases.

Host Setup
==========

The tools in the openstack-on-lxd git repository require the use of Juju 2.x, which provides full support for the LXD local provider.

Ubuntu 18.04 (Bionic) is the current LTS release and should be used for this procedure.

.. code:: bash

    sudo snap install juju --classic

    sudo apt install zfsutils-linux squid-deb-proxy bridge-utils \
        python-novaclient python-keystoneclient python-glanceclient \
        python-neutronclient python-openstackclient curl


You'll need a well specified machine with at least 8G of RAM and a SSD; for reference the author uses Lenovo x240 with an Intel i5 processor, 16G RAM and a 500G Samsung SSD (split into two - one partition for the OS and one partition for a ZFS pool).

For s390x, this has been validated on an LPAR with 12 CPUs, 40GB RAM, 2 ~40GB disks (one disk for the OS and one disk for the ZFS pool).

For arm64, this has been validated on a single Gigabyte R120-T33 with 48 cores, 64GB RAM, 1 240GB SSD (for the operating system), 2 1TB SAS disks (for the ZFS pool).

You'll need to clone the repository with the bundles and configuration for the deployment:

.. code:: bash

    git clone https://github.com/openstack-charmers/openstack-on-lxd.git

All commands in this document assume they are being made from within the local copy of this repo.

LXD
===

Base Configuration
~~~~~~~~~~~~~~~~~~

This type of deployment creates numerous containers on a single host which leads to many thousands of file handles.

Some of the default system thresholds may not be high enough for this use case, potentially leading to issues such as `Too many open files`.

To address this, the host system should be configured according to the LXD production-setup_ guide, specifically the ``/etc/sysctl.conf`` bits:

.. code:: bash

    echo fs.inotify.max_queued_events=1048576 | sudo tee -a /etc/sysctl.conf
    echo fs.inotify.max_user_instances=1048576 | sudo tee -a /etc/sysctl.conf
    echo fs.inotify.max_user_watches=1048576 | sudo tee -a /etc/sysctl.conf
    echo vm.max_map_count=262144 | sudo tee -a /etc/sysctl.conf
    echo vm.swappiness=1 | sudo tee -a /etc/sysctl.conf
    sudo sysctl -p

In order to allow the OpenStack Cloud to function, you'll need to reconfigure the default LXD bridge to support IPv4 networking; is also recommended that you use a fast storage backend such as ZFS on a SSD based block device.  Use the :command:`lxd` provided configuration tool to help do this:

.. code:: bash

    sudo lxd init

The referenced ``config.yaml`` uses an apt proxy to improve installation performance.  The network that you create during the :command:`lxd init` procedure should accomodate that address.  Also ensure that you leave a range of IP addresses free to use for floating IP addresses for OpenStack instances. The following are the values which are used in this example procedure:

    Network and IP: 10.0.8.1/24
    DHCP range: 10.0.8.2 -> 10.0.8.200

Also update the default profile to use Jumbo frames for all network connections into containers:

.. code:: bash

    lxc profile device set default eth0 mtu 9000

This will ensure you avoid any packet fragmentation type problems with overlay networks.

Test out your configuration prior to launching an entire cloud:

.. code:: bash

    lxc launch ubuntu-daily:bionic

This should result in a running container you can exec into and back out of:

.. code:: bash

    lxc exec <container-name> bash
    exit

Juju
====

Bootstrap the Juju Controller
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Prior to deploying the OpenStack on LXD bundle, you'll need to bootstrap a controller to manage your Juju models.

Review the contents of the ``config.yaml`` prior to running the following command and edit as appropriate; this configures some defaults for containers created in the model including setting up things like APT proxy to improve performance of network operations.

.. code:: bash

    juju bootstrap --config config.yaml localhost lxd


Juju Profile Update
~~~~~~~~~~~~~~~~~~~

Juju creates a couple of profiles for the models that it creates by default.  After bootstrapping is complete, update the ``juju-default`` :command:`lxc` profile:

.. code:: bash

    cat lxd-profile.yaml | lxc profile edit juju-default

This will ensure that containers created by LXD for Juju have the correct permissions to run your OpenStack cloud.

Configure a PowerNV (ppc64el) Host
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When deployed directly to metal, the nova-compute charm sets ``smt=off``, as is necessary for libvirt usage.  However, when nova-compute is in a container, the containment prevents :command:`ppc64_cpu` from modifying the host's smt value.  It is necessary to pre-configure the host ``smt`` setting for nova-compute (libvirt + qemu) in ``ppc64el`` scenarios.

.. code:: bash

    sudo ppc64_cpu --smt=off


OpenStack
=========

Deploy
~~~~~~

Deploy OpenStack using one of the provided bundles.

You can watch deployment progress using the :command:`juju status` command.
The time required will depend on your system's resources: memory, CPU, disk
subsystem, and network speed.

.. important::

   The deployment must occur within the ``default`` model. This is to ensure that
   the ``juju-default`` LXD profile is applied to the containers.

amd64, arm64, or ppc64el
++++++++++++++++++++++++

For the amd64, arm64, or ppc64el architectures choose from among the available
combinations of OpenStack release and Ubuntu series.

+-------------------+---------------+-----------------------------------------------+
| OpenStack release | Ubuntu series | Deploy command                                |
+===================+===============+===============================================+
| Mitaka            | Xenial        | ``juju deploy ./bundle-xenial-mitaka.yaml``   |
+-------------------+---------------+-----------------------------------------------+
| Newton            | Xenial        | ``juju deploy ./bundle-xenial-newton.yaml``   |
+-------------------+---------------+-----------------------------------------------+
| Ocata             | Xenial        | ``juju deploy ./bundle-xenial-ocata.yaml``    |
+-------------------+---------------+-----------------------------------------------+
| Pike              | Xenial        | ``juju deploy ./bundle-xenial-pike.yaml``     |
+-------------------+---------------+-----------------------------------------------+
| Queens            | Xenial        | ``juju deploy ./bundle-xenial-queens.yaml``   |
+-------------------+---------------+-----------------------------------------------+
| Queens            | Bionic        | ``juju deploy ./bundle-bionic-queens.yaml``   |
+-------------------+---------------+-----------------------------------------------+
| Rocky             | Bionic        | ``juju deploy ./bundle-bionic-rocky.yaml``    |
+-------------------+---------------+-----------------------------------------------+
| Stein             | Bionic        | ``juju deploy ./bundle-bionic-stein.yaml``    |
+-------------------+---------------+-----------------------------------------------+
| Train             | Bionic        | ``juju deploy ./bundle-bionic-train.yaml``    |
+-------------------+---------------+-----------------------------------------------+
| Train             | Eoan          | ``juju deploy ./bundle-eoan-train.yaml``      |
+-------------------+---------------+-----------------------------------------------+

.. important::

   Train deployments will have Ceph Mimic configured in the bundle until a
   solution has been devised to address the dropping of directory backed OSD
   support in Ceph Nautilus. See bug `GH #72`_.

s390x
+++++

For the s390x architecture choose from among the available combinations of
OpenStack release and Ubuntu series.

+-------------------+---------------+---------------------------------------------------+
| OpenStack release | Ubuntu series | Deploy command                                    |
+===================+===============+===================================================+
| Mitaka            | Xenial        | ``juju deploy ./bundle-xenial-mitaka-s390x.yaml`` |
+-------------------+---------------+---------------------------------------------------+
| Newton            | Xenial        | ``juju deploy ./bundle-xenial-newton-s390x.yaml`` |
+-------------------+---------------+---------------------------------------------------+
| Ocata             | Xenial        | ``juju deploy ./bundle-xenial-ocata-s390x.yaml``  |
+-------------------+---------------+---------------------------------------------------+
| Pike              | Xenial        | ``juju deploy ./bundle-xenial-pike-s390x.yaml``   |
+-------------------+---------------+---------------------------------------------------+
| Queens            | Xenial        | ``juju deploy ./bundle-xenial-queens-s390x.yaml`` |
+-------------------+---------------+---------------------------------------------------+
| Queens            | Bionic        | ``juju deploy ./bundle-bionic-queens-s390x.yaml`` |
+-------------------+---------------+---------------------------------------------------+
| Rocky             | Bionic        | ``juju deploy ./bundle-bionic-rocky-s390x.yaml``  |
+-------------------+---------------+---------------------------------------------------+
| Stein             | Bionic        | ``juju deploy ./bundle-bionic-stein-s390x.yaml``  |
+-------------------+---------------+---------------------------------------------------+
| Train             | Bionic        | ``juju deploy ./bundle-bionic-train-s390x.yaml``  |
+-------------------+---------------+---------------------------------------------------+
| Train             | Eoan          | ``juju deploy ./bundle-eoan-train-s390x.yaml``    |
+-------------------+---------------+---------------------------------------------------+

Using the Cloud
~~~~~~~~~~~~~~~

Check Access
++++++++++++

Once deployment has completed (units should report a ready state in the status output), check that you can access the deployed cloud without issues:

.. code:: bash

    source openrcv3_project
    openstack catalog list
    openstack service list
    openstack network agent list
    openstack volume service list

The openstack client commands should all succeed and you should get a feel as to how the various OpenStack components are deployed in each container.

Upload an image
+++++++++++++++

Before we can boot an instance, we need an image to boot in Glance.

.. note:: 

   If you are using a ZFS backend for this deployment, force-raw-images must be disabled on the nova-compute charm in Pike and later.
   We have made this the default in our bundles - however, be aware that using this setting in a production environment is discouraged as it may have an impact on performance.

For amd64:

.. code:: bash

    curl https://cloud-images.ubuntu.com/xenial/current/xenial-server-cloudimg-amd64-disk1.img | \
        openstack image create --public --container-format=bare --disk-format=qcow2 xenial

For arm64:

.. code:: bash

    curl https://cloud-images.ubuntu.com/xenial/current/xenial-server-cloudimg-arm64-uefi1.img | \
        openstack image create --public --container-format=bare --disk-format=qcow2 --property hw_firmware_type=uefi xenial

For s390x:

.. code:: bash

    curl https://cloud-images.ubuntu.com/xenial/current/xenial-server-cloudimg-s390x-disk1.img | \
        openstack image create --public --container-format=bare --disk-format=qcow2 xenial

For ppc64el:

.. code:: bash

    curl https://cloud-images.ubuntu.com/xenial/current/xenial-server-cloudimg-ppc64el-disk1.img | \
        openstack image create --public --container-format=bare --disk-format=qcow2 xenial


Configure the network - Queens and Later
++++++++++++++++++++++++++++++++++++++++

First, create the 'external' network which actually maps directly to the LXD bridge:

.. code:: bash

    ./neutron-ext-net-ksv3 --network-type flat \
        -g 10.0.8.1 -c 10.0.8.0/24 \
        -f 10.0.8.201:10.0.8.254 ext_net

and then create an internal overlay network for the instances to actually attach to:

.. code:: bash

    ./neutron-tenant-net-ksv3 -p admin -r provider-router \
        -N 10.0.8.1 internal 192.168.20.0/24

Configure the network - Pike and Earlier
++++++++++++++++++++++++++++++++++++++++

First, create the 'external' network which actually maps directly to the LXD bridge:

.. code:: bash

    ./neutron-ext-net --network-type flat \
        -g 10.0.8.1 -c 10.0.8.0/24 \
        -f 10.0.8.201:10.0.8.254 ext_net

and then create an internal overlay network for the instances to actually attach to:

.. code:: bash

    ./neutron-tenant-net -t admin -r provider-router \
        -N 10.0.8.1 internal 192.168.20.0/24


Create a key-pair
+++++++++++++++++

Upload your local public key into the cloud so you can access instances:

.. code:: bash

    openstack keypair create --public-key ~/.ssh/id_rsa.pub mykey


Create Flavors
++++++++++++++

It's safe to skip this for Mitaka.  For Newton and later, there are no pre-populated flavors.  Check if flavors exist, and if not, create them:

.. code:: bash

    openstack flavor list

.. code:: bash

    openstack flavor create --public --ram 512 --disk 1 --ephemeral 0 --vcpus 1 m1.tiny
    openstack flavor create --public --ram 1024 --disk 20 --ephemeral 40 --vcpus 1 m1.small
    openstack flavor create --public --ram 2048 --disk 40 --ephemeral 40 --vcpus 2 m1.medium
    openstack flavor create --public --ram 8192 --disk 40 --ephemeral 40 --vcpus 4 m1.large
    openstack flavor create --public --ram 16384 --disk 80 --ephemeral 40 --vcpus 8 m1.xlarge

Boot an instance
++++++++++++++++

You can now boot an instance on your cloud:

.. code:: bash

    openstack server create --image xenial --flavor m1.small --key-name mykey \
       --wait --nic net-id=$(openstack network list | grep internal | awk '{ print $2 }') \
       openstack-on-lxd-ftw

Attaching a volume
++++++++++++++++++

First, create a volume in cinder:

.. code:: bash

    openstack volume create --size 10 testvolume

then attach it to the instance we just booted in nova:

.. code:: bash

    openstack server add volume openstack-on-lxd-ftw testvolume
    openstack volume show testvolume

The attached volume will be accessible once you login to the instance (see below).  It will need to be formatted and mounted!

Accessing your instance
+++++++++++++++++++++++

In order to access the instance you just booted on the cloud, you'll need to assign a floating IP address to the instance:

.. code:: bash

    openstack floating ip create ext_net
    openstack server add floating ip <uuid-of-instance> <new-floating-ip>

Permit SSH and ping quite liberally, by allowing both on all default security groups. This only needs to be done once on the deployed cloud:

.. code:: bash

    for i in $(openstack security group list | awk '/default/{ print $2 }'); do \
        openstack security group rule create $i --protocol icmp --remote-ip 0.0.0.0/0; \
        openstack security group rule create $i --protocol tcp --remote-ip 0.0.0.0/0 --dst-port 22; \
    done

After running these commands you should be able to access the instance from the LXD host:

.. code:: bash

    ssh ubuntu@<new-floating-ip>

Access the GUIs
===============

The method of accessing the GUI IP addresses varies, depending on where the cloud was deployed.  The "public" addresses of the deployed cloud are actually 10.0.8.x addresses, sitting behind NAT.

OpenStack Dashboard
~~~~~~~~~~~~~~~~~~~

First, find the IP address of the openstack-dashboard unit and admin's password by querying Juju:

.. code:: bash

    juju status openstack-dashboard
    juju run --unit keystone/leader 'leader-get admin_passwd'


Then, choose your adventure:

OpenStack-on-LXD is deployed on your local machine
++++++++++++++++++++++++++++++++++++++++++++++++++

The IP address of the OpenStack Dashboard will be locally accessible.

Adjust and visit the following URL from a browser your local machine:

.. code:: bash

    http://<ip address of openstack-dashboard>/horizon

    domain:  admin_domain
    user:  admin
    password:  ??????????

OpenStack-on-LXD is deployed on a remote machine
++++++++++++++++++++++++++++++++++++++++++++++++

The IP address of the GUI will not be directly accessible. You can forward a TCP port across an existing SSH session, then access the dashboard on your localhost.

In your SSH session, press the ~C key combo to initiate an SSH command console on-the-fly.  Adjust and issue the following command to forward ``localhost:10080`` to ``openstack-dashboard:80`` across that existing SSH session:

.. code:: bash

    -L 10080:<ip address of openstack-dashboard>:80

Then visit the following URL from a browser on your local machine:

.. code:: rest

    http://localhost:10080/horizon

.. code:: bash

    domain:  admin_domain
    user:  admin
    password:  ??????????

Juju GUI
~~~~~~~~

First, find the IP address and credentials for the Juju GUI:

.. code:: bash

    juju gui

Then, choose your adventure:

OpenStack-on-LXD is deployed on your local machine
++++++++++++++++++++++++++++++++++++++++++++++++++

The IP address of the Juju GUI will be locally accessible.

The URL provided should work directly, and it should look something like:

.. code:: rest

    https://10.0.8.x:17070/gui/u/admin/default


OpenStack-on-LXD is deployed on a remote machine
++++++++++++++++++++++++++++++++++++++++++++++++

The IP address of the Juju GUI will not be directly accessible. You can forward a TCP port across an existing SSH session, then access the Juju GUI on your localhost.

In your SSH session, press the ~C key combo to initiate an SSH command console on-the-fly.  Adjust and issue the following command to forward ``localhost:10070`` to ``juju-gui:17070`` across that existing SSH session:

.. code:: bash

    -L 10070:<ip address of juju-gui>:17070

Then visit the following URL from a browser on your local machine:

.. code:: rest

    https://localhost:10070/gui/login/u/admin/default

Switching in a dev charm
========================

Now that you have a running OpenStack deployment on your machine, you can switch in your development changes to one of the charms in the deployment:

.. code:: bash

    juju upgrade-charm --switch <path-to-your-charm> cinder

The charm will be upgraded with your local development changes; alternatively you can update the ``bundle.yaml`` to reference your local charm so that its used from the start of cloud deployment.

Known Limitations
=================

Currently is not possible to run Cinder with iSCSI/LVM based storage under LXD; this limits use of block storage options to those that are 100% userspace, such as Ceph.

.. _production-setup: https://github.com/lxc/lxd/blob/master/doc/production-setup.md

.. BUGS
.. _GH #72: https://github.com/openstack-charmers/openstack-on-lxd/issues/72
