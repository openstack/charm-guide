===============
Getting started
===============

Overview
--------

This tutorial shows how to deploy a test Charmed OpenStack cloud. Its major
software elements are:

* Ubuntu 22.04 LTS (Jammy) for the cloud nodes
* OpenStack 2023.1 (Antelope)
* Ceph Quincy

OpenStack services will include Compute, Network, Block Storage, Object
Storage, Identity, Image, and Dashboard.

Requirements
------------

Knowledge
~~~~~~~~~

The assumed reader skill set is that of an intermediate system administrator,
especially in terms of networking. General familiarity with OpenStack itself is
also recommended.

In particular, you should be acquainted with these concepts:

* `bundles`_
* `constraints`_
* `network spaces`_

Hardware
~~~~~~~~

`MAAS`_ is an integral component of Charmed OpenStack. You will therefore need
a MAAS cluster with at least four nodes:

* 1 x Juju controller node: 4GiB RAM, 2 CPUs, 1 NIC, 1 x 40GiB storage
* 3 x cloud nodes: 16GiB RAM, 4 CPUs, 2 NICs, 2 x 80GiB storage

A single network interface is sufficient on the cloud nodes if an Open vSwitch
bridge is set up in MAAS. Instructions for doing this will be provided later in
the tutorial.

.. note::

   The smaller controller node can be targeted via Juju constraints at
   controller-creation time.

Software
~~~~~~~~

The software versions used in this tutorial are:

* MAAS 3.2.7
* Juju 2.9.42

Client
~~~~~~

All operations will be performed from a lightly-resourced client host that has
connectivity to the MAAS cluster.

On this client you should:

* Install `Juju`_ (the Juju client).

* Create directory ``~/tutorial`` . All the files needed for this tutorial will
  be placed there.

Procedure
---------

The procedure consists of the following steps:

.. toctree::
   :maxdepth: 1

   settings
   overlay
   juju
   deploy
   openstack
   verify
   dashboard

.. LINKS
.. _Juju: https://juju.is/docs/olm/install-juju
.. _MAAS: https://maas.io
.. _bundles: https://juju.is/docs/sdk/manage-bundles
.. _constraints: https://juju.is/docs/olm/constraints
.. _network spaces: https://maas.io/docs/maas-concepts-and-terms-reference#heading--spaces
