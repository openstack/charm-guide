.. _charm_anatomy:

=============
Charm Anatomy
=============

Overview
--------

The new OpenStack charms (charms written in 2016 onwards) are written using the
`reactive framework <https://charmsreactive.readthedocs.io/en/latest/>`__ in Python.
An introduction on the reactive framework and building charms from layers can
be found in the `Getting Started with charm development
<https://docs.jujucharms.com/devel/en/developer-getting-started>`__ .
This guide covers only the new reactive charms.

Configuration Files
-------------------

.. _`layers.yaml`:

layers.yaml
~~~~~~~~~~~

The **src/layers.yaml** file defines what layers and interfaces will be imported
and included in the charm when the charm is built. See the `Openstack Layers`_
section and `OpenStack Interfaces`_ section below.  If additional interfaces or
layers add them to the **includes** list within **src/layers.yaml**.

Below is an example of the layers.yaml for an OpenStack API charm which has a
relation with MongoDB:

.. code:: yaml

    includes: ['layer:openstack-api', 'interface:mongodb']
    options:
      basic:
        use_venv: True
        include_system_packages: True

When the charm is built the openstack-api layer and mongodb interface will
be included in the built charm. The charm will run in a virtual env with
system packages exposed in that virtual env.  See the *Layer Configuration*
section in `Basic Layer README <http://charmsreactive.readthedocs.io/en/latest/layer-basic.html>`__
for more details of the configurable options in a **layers.yaml**

config.yaml
~~~~~~~~~~~

The charm authors guide contains a section on the `config.yaml <https://docs.jujucharms.com/devel/en/charms-config>`__
and is a good place to start.  The config.yaml of the built charm is
constructed from each layer that contains a config.yaml.

metadata.yaml
~~~~~~~~~~~~~

The charm
`metadata.yaml <https://docs.jujucharms.com/devel/en/authors-charm-metadata>`__
describes the charm and how it relates to other charms. This is also
constructed from each layer that defines a metadata.yaml


OpenStack Layers
----------------

Basic Layer
~~~~~~~~~~~

The `Basic Layer <https://github.com/juju-solutions/layer-basic>`__ is the
base layer for all charms built using layers. It provides all of the standard
Juju hooks and runs the charms.reactive.main loop for them. It also bootstraps
the charm-helpers and charms.reactive libraries and all of their dependencies
for use by the charm.

OpenStack Layer
~~~~~~~~~~~~~~~

The `Openstack Layer <https://github.com/openstack/charm-layer-openstack>`__
provides the base OpenStack configuration options, templates, template
fragments and dependencies for authoring OpenStack Charms. Typically this layer
is used for subordinate charms. The openstack-api or openstack-principle layers
are probably more appropriate for principle charms and both of those layers
inherit this one.

This layer includes a wheelhouse to pull in `charms.openstack <https://github.com/openstack/charms.openstack>`__
. See `charms.openstack`_  for more details.


Openstack Principle Layer
~~~~~~~~~~~~~~~~~~~~~~~~~

The `Openstack Principle Layer <https://github.com/openstack/charm-layer-openstack-principle>`__
provides the base layer for OpenStack charms that are intended for
use as principle (rather than subordinate)

Openstack API Layer
~~~~~~~~~~~~~~~~~~~

The `Openstack API Layer <https://github.com/openstack/charm-layer-openstack-api>`__
provides the base layer for OpenStack charms that are will deploy API services,
and provides all of the core functionality for:

- HA (using the hacluster charm)
- SSL (using configuration options or keystone for certificates)
- Juju 2.0 network space support for API endpoints
- Configuration based network binding of API endpoints

It also pulls in interfaces mysql-shared, rabbitmq and keystone which are
common to API charms.

.. _`OpenStack Interfaces`:

OpenStack Interfaces
--------------------

Interfaces define the data exchange between each of the charms. A list of all
available interfaces is available `here <https://github.com/juju/layer-index>`__.
A list of OpenStack specific interfaces can be found `here <https://github.com/openstack?q=charm-interface>`__

The interfaces a charm needs are defines in the `layers.yaml`_. Below is a list
of the typical interfaces needed by different OpenStack charm types:

**API Charm**

- `mysql-shared <https://github.com/openstack/charm-interface-mysql-shared>`__
- `rabbitmq <https://github.com/openstack/charm-interface-rabbitmq>`__
- `keystone <https://github.com/openstack/charm-interface-keystone>`__

**Neutron SDN Plugin**

- `neutron-plugin <https://github.com/openstack/charm-interface-neutron-plugin>`__
- `service-control <https://github.com/openstack/charm-interface-service-control>`__

**Neutron ODL Based SDN Plugin**

- `neutron-plugin <https://github.com/openstack/charm-interface-neutron-plugin>`__
- `service-control <https://github.com/openstack/charm-interface-service-control>`__
- `ovsdb-manager <https://github.com/openstack/charm-interface-ovsdb-manager>`__
- `odl-controller-api <https://github.com/openstack/charm-interface-odl-controller-api>`__

**Neutron API Plugin**

- `neutron-plugin-api-subordinate <https://github.com/openstack/charm-interface-neutron-plugin-api-subordinate>`__
- `service-control <https://github.com/openstack/charm-interface-service-control>`__

.. _`charms.openstack`:

charms.openstack
----------------

The `charms.openstack <https://github.com/openstack/charms.openstack>`__ python
module provides helpers for building layered, reactive OpenStack charms. It is
installed by the `OpenStack Layer <https://github.com/openstack/charm-layer-openstack>`_ .

Defining the Charm
------------------

The charm is defined be extending the OpenStackCharm or OpenStackCharmAPI base
classes in **src/lib/charm/openstack/new_charm_name.py** and overriding the
class attributes as needed.

For example to define a charm for a service called 'new-service':

.. code:: python

    import charms_openstack.charm

    class NewServiceCharm(charms_openstack.charm.OpenStackCharm):

        # The name of the charm (for printing, etc.)
        name = 'new-service'

        # List of packages to install
        packages = ['glance-common']

        # The list of required services that are checked for assess_status
        # e.g. required_relations = ['identity-service', 'shared-db']
        required_relations = ['keystone']

        # A dictionary of:
        # {
        #    'config.file': ['list', 'of', 'services', 'to', 'restart'],
        #    'config2.file': ['more', 'services'],
        # }
        # The files that for the keys of the dict are monitored and if the file
        # changes the corresponding services are restarted
        restart_map = {
            '/etc/new-svc/new-svc.conf': ['new-charm-svc']}

        # first_release = this is the first release in which this charm works
        release = 'icehouse'

        def configure_foo(self):
            ...

The charm definition above can also define methods, like configure_foo, that
the charm handlers can call to run charm specific code.

Reacting to Events
------------------

Reactive charms react to events. These events could be raised by interfaces or
by other handlers. A number of event handlers are added by default by the
`charms.openstack`_ module. For example, an install handler runs by default and
will install the packages which were listed in NewServiceCharm.packages. Once
complete the 'charm.installed' state is raised.  The charms handlers specific
to the new charm are defined in
**src/reactive/new_charm_name_handlers.py**

For example, once the packages are installed it is likely that additional
configuration is needed e.g. rendering config, configuring bridges or updating
remote services via their interfaces. To perform an action once the initial
package installation has been done a handler needs to be added to listen for
the **charm.installed** event. To do this edit
**src/reactive/new_charm_name_handlers.py** and add the reactive handler:

.. code:: python

    @reactive.when('charm.installed')
    def configure_foo():
        with charm.provide_charm_instance() as new_charm:
            new_charm.configure_foo()

If configure_foo() should only be run once then the handler can emit a new
state and the running of configure_foo gated on the state not being present
e.g.

.. code:: python

    @reactive.when_not('foo.configured')
    @reactive.when('charm.installed')
    def configure_foo():
        with charm.provide_charm_instance() as new_charm:
            new_charm.configure_foo()
        reactive.set_state('foo.configured')


File Templates
--------------

Most charms need to write a configuration file from a template. The templates
are stored in **src/templates** see `Templates Directory`_ for more details. The
context used to populate the template has a number of namespaces which are
populated from different sources. Below outlines those namespaces.

.. NOTE::
   Hypens are always automatically converted to underscores in the template
   context.

Template properties from Interfaces
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

By default some interfaces are automatically allocated a namespace within the
template context. Those namespaces are also automatically populated with some
options directly from the interface. For example if a charm is related to
Keystone's `keystone interface <https://github.com/openstack/charm-interface-keystone>`__
then a number of **service\_** variables are set in the
identity\_service namespace. So, charm template could contain the following to
access those variables:

.. code:: python

    [keystone_authtoken]
    www_authenticate_uri = {{ identity_service.service_protocol }}://{{ identity_service.service_host }}:{{ identity_service.service_port }}
    auth_url = {{ identity_service.auth_protocol }}://{{ identity_service.auth_host }}:{{ identity_service.auth_port }}

See the **auto\_accessors** list in `charm-interface-keystone <https://github.com/openstack/charm-interface-keystone/blob/master/requires.py>`__
for a complete list

However, most interface data is accessed via Adapters...

Template properties from Adapters
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Adapters are used to take the data from an interface and create new variables
in the template context. For example the **RabbitMQRelationAdapter** (which can
be found in the `adapters.py <https://github.com/openstack/charms.openstack/blob/master/charms_openstack/adapters.py>`__
from charms.openstack.) adds an **ssl\_ca\_file** variable to the amqp
namespace. This setting is really independent of the interface with rabbit but
should be consistent across the OpenStack deployment. This variable can then
be accessed in the same way as the rest of the amqp setting ``{{amqp.ssl_ca_file }}``

Template properties from user config
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The settings exposed to the user via the config.yaml are added to the
**options** namespace.  The value the user has set for option  **foo** can be
retrieved inside a template by including ``{{ options.foo }}``

Template properties added to user config
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

It is useful to be able to set a property based on examining multiple config
options or examining other aspects of the runtime system. The
**charms_openstack.adapters.config_property** decorator can be used to achieve
this. In the example below if the user has set the boolean config option
**angry** to **True** and set the **radiation** string config option to
**gamma** then the **hulk_mode** property is set to True.

.. code:: python

    @charms_openstack.adapters.config_property
    def hulk_mode(config):
        if config.angry and config.radiation =='gamma':
            return True
        else:
            return False

This can be accessed in the templates with ``{{ options.hulk_mode }}``

Template properties added to an Adapter
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To be able to set a property based on the settings retrieved from an interface.
In the example below the charm sets a pipeline based on the Keystone API
version advertised by the keystone interface,

.. code:: python

    @charms_openstack.adapters.adapter_property('identity_service')
    def charm_pipeline(keystone):
        return {
            "2": "cors keystone_authtoken context apiapp",
            "3": "cors keystone_v3_authtoken context apiapp",
            "none": "cors unauthenticated-context apiapp"
        }[keystone.api_version]

This can be accessed in the templates with ``{{ identity_service.charm_pipeline }}``


.. _`Templates Directory`:

Templates Directory
~~~~~~~~~~~~~~~~~~~

Template are loaded from several places in the following order:

- From the most recent OS release-specific template dir (if one exists)
- Working back through the template directories for each earlier OpenStack Release
- The base templates_dir

For the example above, 'templates' contains the following structure:

::

        templates/nova.conf
        templates/api-paste.ini
        templates/kilo/api-paste.ini
        templates/newton/api-paste.ini

If the charm is deploying the Newton release, it first searches
the newton directory for nova.conf, then the templates dir. So
**templates/nova.conf** will be used.

When writing api-paste.ini, it will find the template in the newton
directory.

However if Liberty was being installed then the charm would fall back to the
kilo template for api-paste.ini since there is no Liberty specific version.

Rendering a Template
~~~~~~~~~~~~~~~~~~~~

Rendering the templates does not usually make sense until all the interfaces
that are going to supply the template context with data are ready and
available. The ``@reactive.when`` decorator not only ensures that the wrapped
method is not run until the interface is ready, it also passes an instance of
the interface to the method it is wrapping. These interfaces can then be passed
to the render_with_interfaces class which looks after finding the templates
and rendering them. render_with_interfaces decides which files need rendering
by examining the keys of the restart_map dict which was specified as part of
the charm class. Taking all this together results in a handler like this:

.. code:: python

    @reactive.when('shared-db.available')
    @reactive.when('identity-service.available')
    @reactive.when('amqp.available')
    def render_config(*args):
        with charm.provide_charm_instance() as new_charm:
            new_charm.render_with_interfaces(args)
            new_charm.assess_status()



Sending data via an Interface
-----------------------------

Some interfaces are used to send as well as receive data. The interface will
expose a method for sending data to a remote application if it is supported.
For example the `neutron-plugin interface <https://github.com/openstack/charm-interface-neutron-plugin>`__
can be used to send configuration to the principle charm.

The handler below waits for the neutron-plugin relation with the principle to
be complete at which point the **neutron-plugin.connected** state will be set
which will fire this trigger. An instance of the interface is passed by the
decorator to the **configure_neutron_plugin** method. This is in turn passed to
the **configure_neutron_plugin** method in the charm class.

.. code:: python

    @reactive.when('neutron-plugin.connected')
    def configure_neutron_plugin(neutron_plugin):
        with charm.provide_charm_instance() as new_charm:
            new_charm.configure_neutron_plugin(neutron_plugin)

In the charm class the instance of the interface is used to update the
principle

.. code:: python

    def configure_neutron_plugin(self, neutron_plugin):
        neutron_plugin.configure_plugin(
            plugin='mysdn',
            config={
                "nova-compute": {
                    "/etc/nova/nova.conf": {
                        "sections": {
                            'DEFAULT': [
                                ('firewall_driver',
                                 'nova.virt.firewall.'
                                 'NoopFirewallDriver'),
                                ('libvirt_vif_driver',
                                 'nova.virt.libvirt.vif.'
                                 'LibvirtGenericVIFDriver'),
                                ('security_group_api', 'neutron'),
                            ],
                        }
                    }
                }
            })

On receiving this data from the neutron_plugin relation the principle will add
the requested config into **/etc/nova/nova.conf**

.. NOTE::
   The amqp, shared-db and identity-service interfaces are automatically
   updated so there is no need to add code for them unless a bespoke
   configuration is needed.


Displaying Charm Status
-----------------------

The charm can declare what state it is in and this status is displayed to the
user via *juju status*. By default the charm code will look for the
``required_relations`` attribute of the charm class. ``required_relations`` is
a list of interfaces. e.g. for an API charm ...

.. code:: python

    required_relations = ['shared-db', 'amqp', 'identity-service']

The in built ``assess_status()`` method will check that each interface has
raised the `{relation}.available` state. If the relation is missing altogether
or if the relation has yet to raise the `{relation}.available` state then a
message is returned via ``juju status``
