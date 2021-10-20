:orphan:

======================================
Scale out the nova-compute application
======================================

Introduction
------------

Scaling out the nova-compute application implies the addition of one or more
nova-compute units (i.e. compute nodes). It is a straightforward operation
that should not incur any cloud downtime.

Procedure
---------

Check the current state of the cloud, scale out by adding a single compute
node, and verify the new node.

Current state
~~~~~~~~~~~~~

Gather basic information about the current state of the cloud in terms of the
nova-compute application:

.. code-block:: none

   juju status nova-compute

Below is sample output from an OVN-based cloud. This example cloud has a single
nova-compute unit:

.. code-block:: console

   Model      Controller  Cloud/Region      Version  SLA          Timestamp
   openstack  maas-one    maas-one/default  2.9.0    unsupported  13:32:43Z

   App           Version  Status  Scale  Charm         Store       Channel  Rev  OS      Message
   ceph-osd               active      0  ceph-osd      charmstore  stable   310  ubuntu  Unit is ready (1 OSD)
   nova-compute  23.0.0   active      1  nova-compute  charmstore  stable   327  ubuntu  Unit is ready
   ntp           3.5      active      1  ntp           charmstore  stable    45  ubuntu  chrony: Ready
   ovn-chassis   20.12.0  active      1  ovn-chassis   charmstore  stable    14  ubuntu  Unit is ready

   Unit              Workload  Agent  Machine  Public address  Ports    Message
   nova-compute/0*   active    idle   0        10.0.0.222               Unit is ready
     ntp/0*          active    idle            10.0.0.222      123/udp  chrony: Ready
     ovn-chassis/0*  active    idle            10.0.0.222               Unit is ready

   Machine  State    DNS         Inst id  Series  AZ       Message
   0        started  10.0.0.222  node1    focal   default  Deployed

Display the name of the current compute host:

.. code-block:: none

   openstack host list

   +---------------------+-----------+----------+
   | Host Name           | Service   | Zone     |
   +---------------------+-----------+----------+
   | juju-616a7f-0-lxd-3 | conductor | internal |
   | juju-616a7f-0-lxd-3 | scheduler | internal |
   | node1.maas          | compute   | nova     |
   +---------------------+-----------+----------+

Scale out
~~~~~~~~~

Use the ``add-unit`` command to scale out the nova-compute application.
Multiple units can be added with the use of the ``-n`` option.

.. note::

   If the node has specific hardware-related requirements (e.g. storage) it
   will need to be manually attended to first (within MAAS) and then targeted
   with the ``--to`` option.

   The new unit can also be placed on an existing Juju machine (co-located with
   another application). In this case, if the ``--to`` option is used it will
   refer to the machine ID.

Here we add a single unit onto a new machine (MAAS node):

.. code-block:: none

   juju add-unit --to node4.maas nova-compute

The status output should eventually look similar to:

.. code-block:: console

   Model      Controller  Cloud/Region      Version  SLA          Timestamp
   openstack  maas-one    maas-one/default  2.9.0    unsupported  14:05:36Z

   App           Version  Status  Scale  Charm         Store       Channel  Rev  OS      Message
   ceph-osd      16.2.0   active      1  ceph-osd      charmstore  stable   310  ubuntu  Unit is ready (1 OSD)
   nova-compute  23.0.0   active      2  nova-compute  charmstore  stable   327  ubuntu  Unit is ready
   ntp           3.5      active      2  ntp           charmstore  stable    45  ubuntu  chrony: Ready
   ovn-chassis   20.12.0  active      2  ovn-chassis   charmstore  stable    14  ubuntu  Unit is ready

   Unit              Workload  Agent  Machine  Public address  Ports    Message
   ceph-osd/0        active    idle   0        10.0.0.222               Unit is ready (1 OSD)
   nova-compute/0*   active    idle   0        10.0.0.222               Unit is ready
     ntp/0*          active    idle            10.0.0.222      123/udp  chrony: Ready
     ovn-chassis/0*  active    idle            10.0.0.222               Unit is ready
   nova-compute/1    active    idle   3        10.0.0.241               Unit is ready
     ntp/1           active    idle            10.0.0.241      123/udp  chrony: Ready
     ovn-chassis/1   active    idle            10.0.0.241               Unit is ready

   Machine  State    DNS         Inst id  Series  AZ       Message
   0        started  10.0.0.222  node1    focal   default  Deployed
   3        started  10.0.0.241  node4    focal   default  Deployed

Verification
~~~~~~~~~~~~

Verify that the new compute node is functional by creating a VM on it.

First confirm that the new compute host is known to the cloud:

.. code-block:: none

   openstack host list

   +---------------------+-----------+----------+
   | Host Name           | Service   | Zone     |
   +---------------------+-----------+----------+
   | juju-616a7f-0-lxd-3 | conductor | internal |
   | juju-616a7f-0-lxd-3 | scheduler | internal |
   | node1.maas          | compute   | nova     |
   | node4.maas          | compute   | nova     |
   +---------------------+-----------+----------+

Then create a VM by targeting the new host, in this case 'node4.maas'. Note
that a minimum Nova API Microversion is required (the cloud admin role is
needed to specify this):

.. code-block:: none

   openstack --os-compute-api-version 2.74 server create \
      --image focal-amd64 --flavor m1.micro --key-name admin-key \
      --network int_net --host node4.maas \
      focal-2

Confirm that the new node is being used (information only available to the
cloud admin by default):

.. code-block:: none

   openstack server show focal-2 | grep hypervisor

   | OS-EXT-SRV-ATTR:hypervisor_hostname | node4.maas
