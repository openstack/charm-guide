.. _new_sdn_charm:

====================
Develop an SDN charm
====================

.. important::

   This page has been identified as being affected by the breaking changes
   introduced between versions 2.9.x and 3.x of the Juju client. Read
   support note :ref:`juju_29_3x_changes` before continuing.

Before writing the charm the charm author needs to have a clear idea of what
applications the charm is going to need to relate to, what files and services
the charm is going to manage and possibly what files or services do other
charms manage that need updating.

In the example below we will assume that a new charm, VirtualTokenRing, is
needed to install a component on compute nodes and to inject some
configuration into nova.conf.

Prerequisites
=============

This will change once the OpenStack templates are on pypi

.. code:: bash

   mkdir sdn-charm
   cd ~/sdn-charm
   git clone git@github.com:openstack-charmers/charm-templates-openstack.git
   cd charm_templates_openstack
   sudo ./setup.py install

Create Charm
============

Charm tools provides a utility for building an initial charm from a template.
During the charm generation charm tools asks a few questions about the charm.

.. code:: bash

    cd ~/sdn-charm
    charm-create  -t openstack-neutron-plugin virtual-token-ring
    INFO: Generating charm for virtual-token-ring in ./virtual-token-ring
    INFO: No virtual-token-ring in apt cache; creating an empty charm instead.
    What is the earliest OpenStack release this charm is compatible with? liberty
    What packages should this charm install (space separated list)?

.. _`build_sdn_charm`:

Build SDN Charm
===============

The charm now needs to be built to pull down all the interfaces and layers the
charm depends on and rolled into the built charm which can be deployed.

.. code:: bash

    cd ~/sdn-charm/virtual-token-ring
    charm build -o build src

Deploy Charm
============

.. code:: bash

    cd build
    juju deploy cs:xenial/nova-compute
    juju deploy ~/sdn-charm/virtual-token-ring/build/virtual-token-ring
    juju integrate nova-compute virtual-token-ring

``juju status`` will now show both charms deployed. The ``nova-compute`` status
will show some missing relations but that's not an issue for this demonstration.


Updating nova.conf
==================

During the initial install of this SDN charm, the standard charms.openstack
default installer will install the packages specified in the class
CharmName.packages, but it will not do any other configuration.
In order to update nova.conf in the nova-compute principal charm, this
virtual-token-ring subordinate charm will need to access the `neutron plugin <https://opendev.org/openstack/charm-interface-neutron-plugin>`__
interface, which will allow it to send configuration information to the
nova-computer principal charm for inclusion in nova.conf on the co-located
machine.


Return to the **virtual-token-ring** directory and edit
**src/reactive/virtual_token_ring_handlers.py**. Add any config that needs
setting in nova.conf.

.. code:: python

    @reactive.when('neutron-plugin.connected')
    def configure_neutron_plugin(neutron_plugin):
        neutron_plugin.configure_plugin(
            plugin='ovs',
            config={
                "nova-compute": {
                    "/etc/nova/nova.conf": {
                        "sections": {
                            'DEFAULT': [
                                ('random_option', 'true'),
                            ],
                        }
                    }
                }
            })

This tells the charm to send that configuration to the principle where the
**neutron-plugin.connected** event has been raised. Then repeat the `build_sdn_charm`_
steps.

Deploy Update
=============

The freshly built charm which contains the update now needs to be deployed to
the environment.

.. code:: bash

    juju upgrade-charm --path ~/sdn-charm/virtual-token-ring/build/virtual-token-ring virtual-token-ring


Check Update
============

.. code:: bash

    juju exec --unit nova-compute/0 "grep random_option /etc/nova/nova.conf"
    random_option = true


