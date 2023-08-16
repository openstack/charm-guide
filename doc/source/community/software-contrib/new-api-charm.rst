.. _new_api_charm:

====================
Develop an API charm
====================

.. important::

   This page has been identified as being affected by the breaking changes
   introduced between versions 2.9.x and 3.x of the Juju client. Read
   support note :ref:`juju_29_3x_changes` before continuing.

The example below will walk through the creation of a basic API charm for the
Openstack `Congress <https://wiki.openstack.org/wiki/Congress>`__ service.
The charm will use prewritten Openstack `layers <https://opendev.org/explore/repos?q=charm-layer>`__
and `interfaces <https://opendev.org/explore/repos?q=charm-interface>`__. Once the charm
is written it will be composed using `charm tools <https://github.com/juju/charm-tools/>`__.
For more details of the internal of a charm see Charm Anatomy.

Before writing a new charm the charm author needs to have a clear idea of what
applications the charm is going to need to relate to, what files and services
the charm is going to manage and possibly what files or services do other
charms manage that need updating.

The Congress service needs to register endpoints with Keystone. It needs a
service username and password and it also needs a MySQL backend to store its
schema.

Create the skeleton charm
=========================

Prerequisites
~~~~~~~~~~~~~

The charm-tools package and charm-templates-openstack python module are both
needed to construct the charm from a template and to build the resulting charm.

.. code:: bash

   sudo apt-get install charm-tools python-jinja2
   mkdir ~/congress-charm
   cd ~/congress-charm
   git clone git@github.com:openstack-charmers/charm-templates-openstack.git
   cd charm-templates-openstack
   sudo ./setup.py install

Create Charm
~~~~~~~~~~~~

Charm tools provides a utility for building an initial charm from a template.
The charm can be thought of as the top layer, the OpenStack layers sit beneath
it and the reactive base layer is at the bottom.

During the charm generation charm tools asks a few questions about the charm.

.. code::

    cd ~/congress-charm
    charm-create  -t openstack-api congress

All the questions are optional, below are the responses for Congress.

.. code::

    What port does the primary service listen on ? 1789
    What is the name of the api service? congress-server
    What type of service is this (used for keystone registration)? congress
    What is the earliest OpenStack release this charm is compatible with? Mitaka
    Where command is used to sync the database? congress-db-manage --config-file /etc/congress/congress.conf upgrade head
    What packages should this charm install (space separated list)? congress-server congress-common python-antlr3 python-pymysql
    List of config files managed by this charm (space separated) /etc/congress/congress.conf
    What is the name of the init script which controls the primary service congress-server

Configuration Files
~~~~~~~~~~~~~~~~~~~

The charm code searches through the templates directories looking for a
directory corresponding to the OpenStack release being installed or earlier.
Since Mitaka is the earliest release the charm is supporting a directory called
Mitaka will house the templates and files.

A template for congress.conf is needed which will have connection
information for MySQL and Keystone as well as user controllable config options.
Create **~/congress-charm/congress/src/templates/mitaka/congress.conf** with
the following contents:

.. code:: bash

    [DEFAULT]
    bind_host = {{ options.service_listen_info.congress_server.ip }}
    bind_port = {{ options.service_listen_info.congress_server.port }}
    auth_strategy = keystone
    drivers = congress.datasources.neutronv2_driver.NeutronV2Driver,congress.datasources.glancev2_driver.GlanceV2Driver,congress.datasources.nova_driver.NovaDriver,congress.datasources.keystone_driver.KeystoneDriver,congress.datasources.ceilometer_driver.CeilometerDriver,congress.datasources.cinder_driver.CinderDriver,congress.datasources.swift_driver.SwiftDriver,congress.datasources.plexxi_driver.PlexxiDriver,congress.datasources.vCenter_driver.VCenterDriver,congress.datasources.murano_driver.MuranoDriver,congress.datasources.ironic_driver.IronicDriver

    [database]
    connection = {{ shared_db.uri }}

    {% include "parts/section-keystone-authtoken" %}



.. _`Build API Charm`:

Build API Charm
~~~~~~~~~~~~~~~

The charm now needs to be built to pull down all the interfaces and layers the
charm depends on and rolled into the built charm which can be deployed.

.. code:: bash

    cd ~/congress-charm/congress
    charm build -o build src

Deploy Charm
~~~~~~~~~~~~

Asumming that an OpenStack cloud is already deployed, add the new Congress
charm.

.. code:: bash

    juju deploy ~/congress-charm/congress/build/congress
    juju integrate congress keystone
    juju integrate congress rabbitmq-server
    juju integrate congress mysql

``juju status`` will show the deployment as it proceeds.

Test Charm
~~~~~~~~~~

.. code:: bash

    $ openstack catalog show congress
    +-----------+---------------------------------------+
    | Field     | Value                                 |
    +-----------+---------------------------------------+
    | endpoints | RegionOne                             |
    |           |   publicURL: http://10.5.3.128:1789   |
    |           |   internalURL: http://10.5.3.128:1789 |
    |           |   adminURL: http://10.5.3.128:1789    |
    |           |                                       |
    | name      | congress                              |
    | type      | policy                                |
    +-----------+---------------------------------------+

    $ openstack congress policy list
    +--------------------------------------+----------------+----------+--------------+-----------------------+
    | id                                   | name           | owner_id | kind         | description           |
    +--------------------------------------+----------------+----------+--------------+-----------------------+
    | 0801bffe-acd0-4644-adab-12321efa0aaf | classification | user     | nonrecursive | default policy        |
    | 38e375ec-b769-45e6-89ad-9eb62da85c57 | action         | user     | action       | default action policy |
    +--------------------------------------+----------------+----------+--------------+-----------------------+


Scaling Out
~~~~~~~~~~~

Another unit can be added to the application to share the workload.

.. code:: bash

    juju add-unit congress

Juju now shows two units of the Congress application.

.. code:: bash

    $ juju status congress --format=oneline
    - congress/1: 10.5.3.128 (agent:idle, workload:active)
    - congress/2: 10.5.3.129 (agent:idle, workload:active)

The charm configures an instance of haproxy on each unit of the application.
Haproxy has all the backends registered within it and load balances traffic
across them.

.. code:: bash

    $ juju ssh congress/1 "tail -11 /etc/haproxy/haproxy.cfg"
    frontend tcp-in_congress-server_admin
        bind \*:1789
        acl net_10.5.3.128 dst 10.5.3.128/255.255.0.0
        use_backend congress-server_admin_10.5.3.128 if net_10.5.3.128
        default_backend congress-server_admin_10.5.3.128

    backend congress-server_admin_10.5.3.128
        balance leastconn
        server congress-2 10.5.3.129:1779 check
        server congress-1 10.5.3.128:1779 check

However, the congress endpoint registered in Keystone is still 10.5.3.128, so
if congress/1 dies clients will fail to connect unless they explicitly set
congress url. To fix this a Congress VIP can be registered in Keystone and
the VIP floated across the Congress units using the hacluster charm.

Adding HA
~~~~~~~~~

The hacluster charm can manage a VIP which is registered with keystone. In
the event of a unit failure the VIP fails over to another application unit and
clients can continue without having to amend their clients.

The congress charm exposes a vip and vip_cidr config options which it passes
to the hacluster charm when the two are joined.

.. code:: bash

    juju deploy hacluster
    juju set-config congress vip=10.5.100.1 vip_cidr=24
    juju integrate hacluster congress

Juju status now reflects the new charms

.. code:: bash

    $ juju status congress --format=oneline

    - congress/1: 10.5.3.128 (agent:idle, workload:active)
      - hacluster/0: 10.5.3.128 (agent:idle, workload:active)
    - congress/2: 10.5.3.129 (agent:idle, workload:active)
      - hacluster/1: 10.5.3.129 (agent:idle, workload:active)

Querying keystone now shows the VIP being used for the congress endpoint, and
the congress client still works unaltered.

.. code:: bash

    $ openstack catalog show congress
    +-----------+---------------------------------------+
    | Field     | Value                                 |
    +-----------+---------------------------------------+
    | endpoints | RegionOne                             |
    |           |   publicURL: http://10.5.100.1:1789   |
    |           |   internalURL: http://10.5.100.1:1789 |
    |           |   adminURL: http://10.5.100.1:1789    |
    |           |                                       |
    | name      | congress                              |
    | type      | policy                                |
    +-----------+---------------------------------------+


    $ openstack congress policy list
    +--------------------------------------+----------------+----------+--------------+-----------------------+
    | id                                   | name           | owner_id | kind         | description           |
    +--------------------------------------+----------------+----------+--------------+-----------------------+
    | 0801bffe-acd0-4644-adab-12321efa0aaf | classification | user     | nonrecursive | default policy        |
    | 38e375ec-b769-45e6-89ad-9eb62da85c57 | action         | user     | action       | default action policy |
    +--------------------------------------+----------------+----------+--------------+-----------------------+


Tidy Up
=======

License File
~~~~~~~~~~~~

The template assumes that the charm will be covered by the `Apache 2.0 License
<https://www.apache.org/licenses/LICENSE-2.0>`__. If another license is to be
used please review the copyright files.

Metadata Description
~~~~~~~~~~~~~~~~~~~~

The `src/metadata.yaml <https://juju.is/docs/sdk/metadata-reference>`__
describes the charm. Update the description and tags in here.


Publish Charm
~~~~~~~~~~~~~

Push charm up to your namespace in the charmstore:

.. code:: bash

    cd ~/congress-charm/congress/build
    charm push . cs:~<lp-usrname>/xenial/congress

To make the charm available to others:

.. code:: bash

    charm grant cs:~<lp-usrname>/xenial/congress everyone
