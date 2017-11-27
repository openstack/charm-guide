================
Charm Deployment
================

In order to test or develop using the OpenStack charms, you'll need to prepare
an environment where you can deploy and test OpenStack services deployed using
charms.

The `OpenStack-on-LXD <openstack-on-lxd.html>`__ procedure provides an
all-in-one machine approach to deploying for the purpose of charm testing,
package exercises, and architecture validation.

The `Deploying from the Charm Store <charm-store.html>`__ section describes
the basics on deploying a bundle or charm from the charm store.

The `Charm Deployment Guide`_ provides a more in-depth guide to deploying a
multi-node OpenStack cloud with Juju, MAAS, and OpenStack Charms.

This part of the Charm guide details methods for getting started in this area,
with anything from a single laptop, through to a full multi-server environment
managed using MAAS.


.. toctree::
   :titlesonly:
   :maxdepth: 1

   openstack-on-lxd
   charm-store
   Charm Deployment Guide <https://docs.openstack.org/charm-deployment-guide/latest/>

.. _Charm Deployment Guide: https://docs.openstack.org/charm-deployment-guide/latest/
