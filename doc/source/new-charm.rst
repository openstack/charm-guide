.. _new_api_charm:

New API Charm
=============

Overview
--------

This guide will walk through the creation of a basic API charm for the Openstack
`Congress <https://wiki.openstack.org/wiki/Congress>`__ service.

The charm will use prewritten Openstack `layers and interfaces <https://github.com/openstack-charmers>`__.

Once the charm is written it will be composed using `charm tools <https://github.com/juju/charm-tools/>`__.

The Congress service needs to register endpoints with Keystone. It needs
a service username and password and it also needs a MySQL backend to
store its schema.

Create the skeleton charm
-------------------------

Firstly create a directory for the new charm and manage the charm with git.

.. code:: bash

    mkdir -p congress/src
    cd congress
    git init

The top layer of this charm is the Congress specific code this code will live in the charm subdirectory.


.. code:: bash

    mkdir -p src/{reactive,lib/charm/openstack}

Describe the Service and required layer(s)
------------------------------------------

The new charm needs a basic src/metadata.yaml to describe what service the charm provides. Edit src/metadata.yaml

.. code:: yaml

    name: congress
    summary: Policy as a service
    description: |
     Congress is an open policy framework for the cloud. With Congress, a cloud
     operator can declare, monitor, enforce, and audit "policy" in a heterogeneous
     cloud environment.

The `openstack-api layer <https://github.com/openstack-charmers/charm-layer-openstack-api>`__
defines a series of config options and interfaces which are mostly common across Openstack
API services e.g. including the openstack-api-layer will pull in the Keystone and MySQL
interfaces (among others) as well as the charm layers the new Congress charm can
leverage.

To instruct "charm build" to pull in the openstack-api layer edit src/layer.yaml:

.. code:: yaml

    includes: ['layer:openstack-api']

Add Congress configuration
--------------------------

Define Congress attributes
~~~~~~~~~~~~~~~~~~~~~~~~~~

There is a base OpenStackCharm class which provides the skeleton for the charm.
Creating a child class from OpenStackCharm allows Congress specific attributes
to be set, like which packages to install, which config files need rendering
etc. This is all done in the src/lib/charm/openstack/congress.py file.

.. code:: python

    import charmhelpers.contrib.openstack.utils as ch_utils

    import charms_openstack.charm
    import charms_openstack.adapters
    import charms_openstack.ip as os_ip

    class CongressCharm(charms_openstack.charm.OpenStackCharm):

        service_name = 'congress'
        release = 'mitaka'

        # Packages the service needs installed
        packages = ['congress-server', 'congress-common', 'python-antlr3',
                    'python-pymysql']

        # Init services the charm manages
        services = ['congress-server']

        # Standard interface adapters class to use.
        adapters_class = charms_openstack.adapters.OpenStackRelationAdapters

        # Ports that need exposing.
        default_service = 'congress-api'
        api_ports = {
            'congress-api': {
                os_ip.PUBLIC: 1789,
                os_ip.ADMIN: 1789,
                os_ip.INTERNAL: 1789,
            }
        }

        # Database sync command used to initalise the schema.
        sync_cmd = ['congress-db-manage', '--config-file',
                    '/etc/congress/congress.conf', 'upgrade', 'head']

        # The restart map defines which services should be restarted when a given
        # file changes
        restart_map = {
            '/etc/congress/congress.conf': ['congress-server'],
            '/etc/congress/api-paste.ini': ['congress-server'],
            '/etc/congress/policy.json': ['congress-server'],
        }

        def __init__(self, release=None, **kwargs):
            """Custom initialiser for class
            If no release is passed, then the charm determines the release from the
            ch_utils.os_release() function.
            """
            if release is None:
                release = ch_utils.os_release('python-keystonemiddleware')
            super(CongressCharm, self).__init__(release=release, **kwargs)

        def install(self):
            """Customise the installation, configure the source and then call the
            parent install() method to install the packages
            """
            self.configure_source()
            # and do the actual install
            super(CongressCharm, self).install()

For reasons methods are needed to wrap the calls to the Congress charms class
methods. These can be appended to the bottom of the
src/lib/charm/openstack/congress.py file.

.. code:: python

    def install():
        """Use the singleton from the CongressCharm to install the packages on the
        unit
        """
        CongressCharm.singleton.install()


    def restart_all():
        """Use the singleton from the CongressCharm to restart services on the
        unit
        """
        CongressCharm.singleton.restart_all()


    def db_sync():
        """Use the singleton from the CongressCharm to run db migration
        """
        CongressCharm.singleton.db_sync()


    def setup_endpoint(keystone):
        """When the keystone interface connects, register this unit in the keystone
        catalogue.
        """
        charm = CongressCharm.singleton
        keystone.register_endpoints(charm.service_name,
                                    charm.region,
                                    charm.public_url,
                                    charm.internal_url,
                                    charm.admin_url)


    def render_configs(interfaces_list):
        """Using a list of interfaces, render the configs and, if they have
        changes, restart the services on the unit.
        """
        CongressCharm.singleton.render_with_interfaces(interfaces_list)


    def assess_status():
        """Just call the CongressCharm.singleton.assess_status() command to update
        status on the unit.
        """
        CongressCharm.singleton.assess_status()

Add Congress code to react to events
------------------------------------

Install Congress Packages
~~~~~~~~~~~~~~~~~~~~~~~~~

The reactive framework is going to emit events that the Congress charm can react
to. The charm needs to define how its going to react to these events and also
raise new events as needed.

The first action a charm needs to do is to install the Congress code. This is
by done running the install method from CongressCharm created earlier.

Edit src/reactive/handlers.py.

.. code:: python

    import charms.reactive as reactive
    import charmhelpers.core.hookenv as hookenv

    # This charm's library contains all of the handler code associated with
    # congress
    import charm.openstack.congress as congress


    # use a synthetic state to ensure that it get it to be installed independent of
    # the install hook.
    @reactive.when_not('charm.installed')
    def install_packages():
        congress.install()
        reactive.set_state('charm.installed')

Configure Congress Relation
~~~~~~~~~~~~~~~~~~~~~~~~~~~

At this point the charm could be built and deployed and it would deploy a unit,
and install congress. However there is no code to specify how this charm should
interact with the services it depend on. For example when joining the database
the charm needs to specify the user and database it requires. The following code
configures the relations with the dependant services.

.. note:: ``assess_status()``: when a relation changes the workload
          status may be changed.  e.g. if a interface is complete in the sense
          that it is connected and all information is available, then that
          interface will set the `{relation}.available` (by convention).
          Thus the workload status could change to 'waiting' from 'blocked'.

Append to src/reactive/handlers.py:

.. code:: python

    @reactive.when('amqp.connected')
    def setup_amqp_req(amqp):
        """Use the amqp interface to request access to the amqp broker using our
        local configuration.
        """
        amqp.request_access(username='congress',
                            vhost='openstack')
        congress.assess_status()


    @reactive.when('shared-db.connected')
    def setup_database(database):
        """On receiving database credentials, configure the database on the
        interface.
        """
        database.configure('congress', 'congress', hookenv.unit_private_ip())
        congress.assess_status()


    @reactive.when('identity-service.connected')
    def setup_endpoint(keystone):
        congress.setup_endpoint(keystone)
        congress.assess_status()

Configure Congress
------------------

Now that the charm has the relations defined that it needs the Congress charm
is in a postion to generate its configuration files.

Create templates
~~~~~~~~~~~~~~~~

The charm code searches through the templates directories looking for a directory
corresponding to the Openstack release being installed or earlier. Since Mitaka
is the earliest release the charm is supporting a directory called mitaka will
house the templates and files.

.. code:: bash

    ( cd /tmp; apt-get source congress-server; )
    mkdir -p templates/mitaka
    cp /tmp/congress*/etc/{api-paste.ini,policy.json} templates/mitaka

A template for congress.conf is needed which will have have connection
information for MySQL, RabbitMQ and Keystone as well as user controllable
config options

.. code:: bash

    [DEFAULT]
    auth_strategy = keystone
    drivers = congress.datasources.neutronv2_driver.NeutronV2Driver,congress.datasources.glancev2_driver.GlanceV2Driver,congress.datasources.nova_driver.NovaDriver,congress.datasources.keystone_driver.KeystoneDriver,congress.datasources.ceilometer_driver.CeilometerDriver,congress.datasources.cinder_driver.CinderDriver,congress.datasources.swift_driver.SwiftDriver,congress.datasources.plexxi_driver.PlexxiDriver,congress.datasources.vCenter_driver.VCenterDriver,congress.datasources.murano_driver.MuranoDriver,congress.datasources.ironic_driver.IronicDriver

    [database]
    connection = {{ shared_db.uri }}

    [keystone_authtoken]
    {% if identity_service.auth_host -%}
    auth_uri = {{ identity_service.service_protocol }}://{{
    identity_service.service_host }}:{{ identity_service.service_port }}
    auth_url = {{ identity_service.auth_protocol }}://{{ identity_service.auth_host
    }}:{{ identity_service.auth_port }}
    auth_plugin = password
    project_domain_id = default
    user_domain_id = default
    project_name = {{ identity_service.service_tenant }}
    username = {{ identity_service.service_username }}
    password = {{ identity_service.service_password }}
    {% endif -%}

Render the config
~~~~~~~~~~~~~~~~~

Now the templates and interfaces are in place the configs can be
rendered. A side-effect of rendering the configs is that any associated
services are restarted. Finally, set the config.complete state this
will be used later to trigger other events.

Append to charm/reactive/handlers.py

.. code:: python

    @reactive.when('shared-db.available')
    @reactive.when('identity-service.available')
    @reactive.when('amqp.available')
    def render_stuff(*args):
        congress.render_configs(args)
        reactive.set_state('config.complete')

Run DB Migration
~~~~~~~~~~~~~~~~

The DB migration can only be run once the config files are in place
since as congress.conf will contain the DB connection information.

To achieve this the DB migration is gated on the config.complete
being set. Finally set the db.synched event so that this is only
run once.

Append to src/reactive/handlers.py

.. code:: python

    @reactive.when('config.complete')
    @reactive.when_not('db.synced')
    def run_db_migration():
        congress.db_sync()
        congress.restart_all()
        reactive.set_state('db.synced')
        congress.assess_status()

Build and Deploy charm
----------------------

Build the charm to pull down the interfaces and layers.

.. code:: bash

    mkdir build
    charm build -obuild src

The built charm can now be deployed with Juju.

.. code:: bash

    juju deploy <full path>/build/congress
    juju add-relation congress mysql
    juju add-relation congress keystone
    juju add-relation congress rabbitmq-server

Deploying an existing Openstack environment is not covered here.
