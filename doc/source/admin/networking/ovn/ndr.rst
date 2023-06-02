====================================
Dynamic Routing with Neutron and OVN
====================================

OpenStack provides a project called `neutron-dynamic-routing`_ (NDR) for the
purposes of advertising network routes and floating IPs created on the
Neutron side to external peers.

.. note::

   As of the OpenStack 2023.1 (Antelope) release, Neutron has enabled NDR
   functionality `with OVN`_.

.. caution::

   There are significant operational considerations to take into account due to
   the presence of `known caveats`_ around how this feature is exposed via
   "ML2/OVN" compared to how it works with "ML2/OVS". This may change in
   the future which is important to take into account for future maintenance
   and upgrades when using this feature.

The following bundle provides an example of a minimal set of applications and
relations necessary for NDR to be able to advertise network or floating IP
routes using BGP speakers according to the `OpenStack NDR documentation`_:

.. code-block:: yaml

   applications:
     keystone-mysql-router:
       charm: ch:mysql-router
       channel: latest/stable
     neutron-api-mysql-router:
       charm: ch:mysql-router
       channel: latest/stable
     mysql-innodb-cluster:
       charm: ch:mysql-innodb-cluster
       num_units: 3
       channel: latest/stable
     keystone:
       charm: ch:keystone
       num_units: 1
       channel: latest/stable
     neutron-api:
       charm: ch:neutron-api
       num_units: 1
       options:
         neutron-security-groups: true
       channel: latest/stable
     neutron-dynamic-routing:
       charm: ch:neutron-dynamic-routing
       num_units: 1
       channel: latest/stable
     rabbitmq-server:
       charm: ch:rabbitmq-server
       num_units: 1
       channel: latest/stable
     ovn-central:
       charm: ch:ovn-central
       num_units: 3
       channel: latest/stable
     neutron-api-plugin-ovn:
       charm: ch:neutron-api-plugin-ovn
       channel: latest/stable
     vault:
       charm: ch:vault
       num_units: 1
       channel: latest/stable
   relations:
     - - neutron-dynamic-routing:amqp
       - rabbitmq-server:amqp
     - - keystone:shared-db
       - keystone-mysql-router:shared-db
     - - keystone-mysql-router:db-router
       - mysql-innodb-cluster:db-router
     - - neutron-api:shared-db
       - neutron-api-mysql-router:shared-db
     - - neutron-api-mysql-router:db-router
       - mysql-innodb-cluster:db-router
     - - neutron-api:amqp
       - rabbitmq-server:amqp
     - - neutron-api:identity-service
       - keystone:identity-service
     - - ovn-central:certificates
       - vault:certificates
     - - ovn-central:ovsdb-cms
       - neutron-api-plugin-ovn:ovsdb-cms
     - - vault:certificates
       - keystone:certificates
     - - vault:certificates
       - neutron-api:certificates
     - - vault:certificates
       - neutron-api-plugin-ovn:certificates
     - - neutron-api-plugin-ovn:neutron-plugin
       - neutron-api:neutron-plugin-api-subordinate

.. LINKS
.. _neutron-dynamic-routing: https://docs.openstack.org/neutron-dynamic-routing/latest/
.. _with OVN: https://github.com/openstack/neutron/commit/4d1a7bd0bc3b142a6dc7f8414ed0d30e6c159057
.. _known caveats: https://bugs.launchpad.net/neutron/+bug/2022058
.. _OpenStack NDR documentation: https://docs.openstack.org/neutron/latest/admin/config-bgp-dynamic-routing.html