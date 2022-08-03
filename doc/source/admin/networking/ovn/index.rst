==========================
Open Virtual Network (OVN)
==========================

Overview
--------

Open Virtual Network (OVN) is an SDN platform. When used with OpenStack the
overall solution is known as "Neutron ML2+OVN". OVN extends the existing
capabilities of a solution based solely on Open vSwitch, which is known as
"Neutron ML2+OVS".

OVN is implemented via a suite of charms:

* neutron-api-plugin-ovn
* ovn-central
* ovn-chassis (or ovn-dedicated-chassis)

.. note::

   The OpenStack Charms project supports OVN starting with OpenStack Train, and
   uses it by default starting with OpenStack Ussuri.

   Instructions for migrating non-OVN clouds to OVN are found on the
   :doc:`../../../project/procedures/ovn-migration` page.

   Due to `feature gaps with ML2+OVS`_, the OpenStack Charms project continues
   to support ML2+OVS.

Deployment
----------

.. important::

   OVN is typically deployed alongside other core components via a
   comprehensive cloud bundle. For example, see the `openstack-base bundle`_.

The below overlay bundle encapsulates what is needed in terms of the
deployment.

.. important::

   An overlay's parameters should be adjusted as per the local environment
   (e.g. the machine mappings). In particular, the following placeholders must
   be replaced with actual values:

   * ``$SERIES``
   * ``$OPENSTACK_ORIGIN``
   * ``$CHANNEL_OVN``

   Replace ``$SERIES`` with the Ubuntu release running on the cloud nodes (e.g.
   'jammy'). For ``$OPENSTACK_ORIGIN`` see the corresponding charm options.
   For channel information see the :doc:`../../../project/charm-delivery` page.

.. code-block:: yaml

   series: $SERIES

   machines:
     '0':
     '1':
     '2':

   relations:
   - - neutron-api-plugin-ovn:certificates
     - vault:certificates
   - - neutron-api-plugin-ovn:neutron-plugin
     - neutron-api:neutron-plugin-api-subordinate
   - - neutron-api-plugin-ovn:ovsdb-cms
     - ovn-central:ovsdb-cms
   - - ovn-central:certificates
     - vault:certificates
   - - ovn-chassis:ovsdb
     - ovn-central:ovsdb
   - - ovn-chassis:certificates
     - vault:certificates
   - - ovn-chassis:nova-compute
     - nova-compute:neutron-plugin

   applications:

     neutron-api:
       options:
         manage-neutron-plugin-legacy-mode=false

     neutron-api-plugin-ovn
       charm: ch:neutron-api-plugin-ovn
       channel: $CHANNEL_OVN

     ovn-central
       charm: ch:ovn-central
       channel: $CHANNEL_OVN
       num_units: 3
       options:
         source: $OPENSTACK_ORIGIN
       to:
       - '0'
       - '1'
       - '2'

     ovn-chassis
       charm: ch:ovn-chassis
       channel: $CHANNEL_OVN

TLS and Vault
~~~~~~~~~~~~~

With the OpenStack charms, OVN requires Vault, which is the chosen software for
managing the TLS certificates that secure control plane communication. This is
achieved via the ``ovn-chassis:certificates vault:certificates`` relation (as
shown in the overlay).

For certificate management information see the :doc:`../../security/tls` page.

See the `vault charm`_ for details on Vault itself.

Data plane
~~~~~~~~~~

The OVN components used for the data plane are deployed by the ovn-chassis
subordinate charm, in conjunction with the nova-compute principal charm. This
is achieved via the ``ovn-chassis:nova-compute nova-compute:neutron-plugin``
relation (as shown in the overlay).

To obtain a dedicated software gateway, the data plane components should be
deployed with the principal `ovn-dedicated-chassis charm`_.

High availability
~~~~~~~~~~~~~~~~~

OVN is natively HA. See the :ref:`OVN section <ha_ovn>` of the Infrastructure
high availability page.

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

Create networks, routers, and subnets through the OpenStack API or CLI as you
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
.. _vault charm: https://charmhub.io/vault
.. _ovn-dedicated-chassis charm: https://charmhub.io/ovn-dedicated-chassis
.. _neutron-api charm: https://charmhub.io/neutron-api
.. _neutron-api-plugin-ovn charm: https://charmhub.io/neutron-api-plugin-ovn
.. _networking-ovn plugin: https://docs.openstack.org/networking-ovn/latest/
.. _feature gaps with ML2+OVS: https://docs.openstack.org/neutron/latest/ovn/gaps.html
.. _Toward Convergence of ML2+OVS+DVR and OVN: http://specs.openstack.org/openstack/neutron-specs/specs/ussuri/ml2ovs-ovn-convergence.html
.. _openstack-base bundle: https://github.com/openstack-charmers/openstack-bundles/blob/master/stable/openstack-base/bundle.yaml
