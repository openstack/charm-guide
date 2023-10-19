=======
Watcher
=======

Overview
--------

`Watcher`_ provides a flexible and scalable resource optimization service for
multi-tenant OpenStack-based clouds. This document describe deploying Charmed
OpenStack with Watcher. It is important to have `Upstream Watcher
Documentation`_ as context.

.. note::

   Watcher is supported by Charmed OpenStack starting with OpenStack 2023.2
   (Bobcat) on Ubuntu 22.04 LTS (Jammy).

There is one Watcher charm that encapsulates all the daemons to provide the
service:

* watcher

The watcher charm provides the `Watcher API`_, `Decision Engine`_ and
`Applier`_.

Deployment
----------

See `Example Bundle`_ for a known working configuration. The bundle deploys the
'watcher' application related to the services needed.

.. important::

   An overlay's parameters should be adjusted as per the local environment
   (e.g. the machine mappings). In particular, the following placeholders must
   be replaced with actual values:

   * ``$SERIES``
   * ``$CHANNEL_OPENSTACK``

   Replace ``$SERIES`` with the Ubuntu release running on the cloud nodes (e.g.
   'jammy'). For channel information see the :doc:`../../project/charm-delivery`
   page.

.. code-block:: yaml

   series: $SERIES
   watcher-mysql-router:
     charm: ch:mysql-router
     channel: 8.0/stable
   watcher:
     charm: ch:watcher
     channel: $CHANNEL_OPENSTACK
     num_units: 1
     options:
       debug: True
       datasources: gnocchi
       planner: weight
       planner-config: >
         {
             "weights": "change_node_power_state:9,change_nova_service_state:50,migrate:30,nop:70,resize:20,sleep:40,turn_host_to_acpi_s3_state:10,volume_migrate:60",
             "parallelization": "change_node_power_state:2,change_nova_service_state:1,migrate:2,nop:1,resize:2,sleep:1,turn_host_to_acpi_s3_state:2,volume_migrate:2"
         }

   relations:
   - - 'watcher:shared-db'
     - 'watcher-mysql-router:shared-db'
   - - 'watcher-mysql-router:db-router'
     - 'mysql-innodb-cluster:db-router'
   - - 'keystone:identity-service'
     - 'watcher:identity-service'
   - - 'rabbitmq-server:amqp'
     - 'watcher:amqp'

Configuration
~~~~~~~~~~~~~

This configuration demonstrates the creation of a optimization strategy to
consolidate the instances running in the less amount ot hypervisor to achieve a
reduction in the energy consumption.

Create an audit template:

.. code-block:: none

   openstack optimize audittemplate create at1 \
       --stategy vm_workload_consolidation \
       --goal server_consolidation

Create an audit:

.. code-block:: none

   openstack optimize audit create \
       --name audit1
       --audit-template at1 \
       --audit-type oneshot \
       --parameter period=600 \
       --parameter granularity=300

Check the action plan:

.. code-block:: none

   openstack optimize actionplan show audit1

Check the actions associated to the plan:

.. code-block:: none

   openstack optimize action list --audit audit1

For more details on the available strategies and their different options see
`Stategies in the Watcher documentation`_.

.. LINKS
.. _Watcher: https://opendev.org/openstack/watcher
.. _Watcher API: https://docs.openstack.org/watcher/latest/architecture.html#watcher-api
.. _Decision Engine: https://docs.openstack.org/watcher/latest/architecture.html#watcher-decision-engine
.. _Applier: https://docs.openstack.org/watcher/latest/architecture.html#watcher-applier
.. _Upstream Watcher Documentation: https://docs.openstack.org/watcher/latest/
.. _Example Bundle: https://opendev.org/openstack/charm-watcher/src/branch/stable/2023.2/src/tests/bundles/jammy-bobcat.yaml
.. _Stategies in the Watcher documentation: https://docs.openstack.org/watcher/latest/strategies/index.html
