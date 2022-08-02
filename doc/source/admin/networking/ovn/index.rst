==========================
Open Virtual Network (OVN)
==========================

Overview
--------

Open Virtual Network (OVN) can be deployed to provide networking services as
part of an OpenStack cloud.

.. note::

   There are feature `gaps from ML2/OVS`_ and deploying legacy ML2/OVS with
   the OpenStack Charms is still available.

OVN charms:

* neutron-api-plugin-ovn
* ovn-central
* ovn-chassis
* ovn-dedicated-chassis

.. note::

   OVN is supported by Charmed OpenStack starting with OpenStack Train. OVN is
   the default configuration in the `OpenStack Base bundle`_ reference
   implementation.

   Instructions for migrating legacy clouds to OVN are found on the
   :doc:`cdg:ovn-migration` page in the Deploy Guide.

Deployment
----------

OVN makes use of Public Key Infrastructure (PKI) to authenticate and authorize
control plane communication. The charm requires a Certificate Authority to be
present in the model as represented by the ``certificates`` relation.
Certificates must be managed by Vault.

.. note::

   For Vault deployment instructions see the `vault charm`_. For certificate
   management information read the :doc:`../../security/tls` page.

To deploy OVN:

.. code-block:: none

   juju config neutron-api manage-neutron-plugin-legacy-mode=false

   juju deploy neutron-api-plugin-ovn
   juju deploy ovn-central -n 3 --config source=cloud:bionic-ussuri
   juju deploy ovn-chassis

   juju add-relation neutron-api-plugin-ovn:certificates vault:certificates
   juju add-relation neutron-api-plugin-ovn:neutron-plugin \
       neutron-api:neutron-plugin-api-subordinate
   juju add-relation neutron-api-plugin-ovn:ovsdb-cms ovn-central:ovsdb-cms
   juju add-relation ovn-central:certificates vault:certificates
   juju add-relation ovn-chassis:ovsdb ovn-central:ovsdb
   juju add-relation ovn-chassis:certificates vault:certificates
   juju add-relation ovn-chassis:nova-compute nova-compute:neutron-plugin

The OVN components used for the data plane is deployed by the ovn-chassis
subordinate charm. A subordinate charm is deployed together with a principle
charm, nova-compute in the example above.

If you require a dedicated software gateway you may deploy the data plane
components as a principle charm through the use of the
`ovn-dedicated-chassis charm`_.

.. note::

   For a concrete example take a look at the `OpenStack Base bundle`_.

High availability
-----------------

OVN is HA by design; take a look at the `OVN section of the Infrastructure high
availability`_ page.

Configuration
-------------

OVN integrates with OpenStack through the OVN ML2 driver. On OpenStack Ussuri
and onwards the OVN ML2 driver is maintained as an in-tree driver in Neutron.
On OpenStack Train it is maintained separately as per the `networking-ovn
plugin`_.

General Neutron configuration is still done through the `neutron-api charm`_,
and the subset of configuration specific to OVN is done through the
`neutron-api-plugin-ovn charm`_.

Usage
-----

Create networks, routers and subnets through the OpenStack API or CLI as you
normally would.

The OVN ML2 driver will translate the OpenStack network constructs into high
level logical rules in the OVN Northbound database.

The ``ovn-northd`` daemon in turn translates this into data in the Southbound
database.

The local ``ovn-controller`` daemon on each chassis consumes these rules and
programs flows in the local Open vSwitch database.

Specific topics on OVN usage are given below:

.. toctree::
   :maxdepth: 1

   hardware-offload
   sriov
   dpdk
   internal-dns
   external-connect
   queries

.. LINKS
.. _vault charm: https://jaas.ai/vault/
.. _Toward Convergence of ML2+OVS+DVR and OVN: http://specs.openstack.org/openstack/neutron-specs/specs/ussuri/ml2ovs-ovn-convergence.html
.. _ovn-dedicated-chassis charm: https://jaas.ai/u/openstack-charmers/ovn-dedicated-chassis/
.. _networking-ovn plugin: https://docs.openstack.org/networking-ovn/latest/
.. _neutron-api charm: https://jaas.ai/neutron-api/
.. _neutron-api-plugin-ovn charm: https://jaas.ai/u/openstack-charmers/neutron-api-plugin-ovn/
.. _OpenStack Base bundle: https://github.com/openstack-charmers/openstack-bundles/tree/master/development/openstack-base-bionic-ussuri-ovn
.. _gaps from ML2/OVS: https://docs.openstack.org/neutron/latest/ovn/gaps.html
.. _OVN section of the Infrastructure high availability: https://docs.openstack.org/charm-guide/latest/admin/ha.html#ovn
