===============
Getting started
===============

Overview
--------

This tutorial shows how to deploy a test Charmed OpenStack cloud. It's major
elements are:

* Ubuntu 20.04 LTS (Focal) for the cloud nodes
* OpenStack Yoga
* Ceph Quincy

OpenStack services will include Compute, Network, Block Storage, Object
Storage, Identity, Image, and Dashboard.

Requirements
------------

Hardware
~~~~~~~~

You will need a `MAAS`_ cluster with four nodes:

* 1 x Juju controller node: 4GiB RAM, 2 CPUs, 1 NIC, 1 x 40GiB storage
* 3 x cloud nodes: 16GiB RAM, 4 CPUs, 2 NICs, 2 x 80GiB storage

.. note::

   The smaller controller node can be targeted via Juju constraints at
   controller-creation time.

   A single network interface is sufficient on the cloud nodes if an Open
   vSwitch bridge is set up in MAAS. See the :doc:`MAAS page
   <cdg:install-maas>` in the Deploy Guide for details.

Software
~~~~~~~~

The software versions used in this tutorial are:

* MAAS 2.9.2
* Juju 2.9.29

Other prerequisites
~~~~~~~~~~~~~~~~~~~

* You should have `Juju`_ installed and be comfortable with its basic usage.

* Create directory ``~/tutorial``. All the files created in this tutorial will
  be placed there.

* The MAAS server must have Focal amd64 images available.

* You should be briefly acquainted with these concepts:

  * `bundles`_
  * `constraints`_
  * `network spaces`_

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
.. _Juju: https://juju.is
.. _MAAS: https://maas.io
.. _bundles: https://jaas.ai/docs/sdk/manage-bundles
.. _constraints: https://juju.is/docs/olm/constraints
.. _network spaces: https://maas.io/docs/snap/3.1/ui/concepts-and-terms#heading--spaces
