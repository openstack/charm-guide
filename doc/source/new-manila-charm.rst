.. _new_manila_charm:

=====================================
New Manila Plugin Configuration Charm
=====================================

Typically, a Manila backend configuration simply needs to provide one or more
configuration sections to the ``manila.conf`` file.  It might also need to
install some additional software to the node that manila is running on. (Note
that, at present, all the manila services run on the same machine unit.)

In order to make it easier to write a configuration charm for a backend, a
template has been provided to bootstrap the process.  This example shows how to
get started and provides pointers to help understand what to do next.

In the example below we will assume that a new charm provides a simple
configuration to the manila charm.

Prerequisites
=============

(Note. This will change once the OpenStack templates are on pypi)

In order get the OpenStack charm templates installed into the ``charm create``
command, the ``charm-templates-openstack`` module needs to be installed
alongside the ``charm-tools`` module.  The code assumes that the
``charm-tools`` module is already installed.

.. code:: bash

   mkdir working
   cd working
   git clone git@github.com:openstack-charmers/charm-templates-openstack.git
   cd charm_templates_openstack
   sudo ./setup.py install

Create Charm
============

Charm tools provides a utility for building an initial charm from a template.
During the charm generation charm tools asks a few questions about the charm.

Note that the charm created from the template is **not** functional and needs
further editing to produce the functional charm needed.

.. code:: bash

    cd path.to.working
    charm-create  -t openstack-manila-plugin new-manila-plugin
    INFO: Generating charm for new-manila-plugin in ./new-manila-plugin
    INFO: No new-manila-plugin in apt cache; creating an empty charm instead.
    What is the earliest OpenStack release this charm will support? mitaka
    What packages should this charm install (space seperated list)?
    What is the package to take the version from (manila-api is probably ok)?

Three questions are asked:

1. The first is about the earliest version of OpenStack that is supported.  The
   ``src/metadata.yaml`` file **must** also be checked/changed to support the
   versions of Ubuntu that the charm will support (e.g. xenial, yakkety, etc.)

2. The second is whether any additional packages (over and above the manila
   packages) should be installed.  Normally, this is just left empty.

3. The third is about where the charm should try and obtain the version from
   (this is the package).  Normally, the driver version will be the same as the
   manila version, and therefore the default is acceptable.  If the driver
   installs further packages, then the version should probably be from one of
   those packages.

Exploring the skeleton charm
============================

The files that the template creates is:

.. code:: bash

    tree
    .
    ├── LICENSE
    ├── requirements.txt
    ├── src
    │   ├── config.yaml
    │   ├── copyright
    │   ├── icon.svg
    │   ├── layer.yaml
    │   ├── lib
    │   │   └── charm
    │   │       └── openstack
    │   │           ├── __init__.py
    │   │           └── new_manila_plugin.py
    │   ├── metadata.yaml
    │   ├── reactive
    │   │   └── new_manila_plugin_handlers.py
    │   ├── README.md
    │   ├── templates
    │   │   └── mitaka
    │   │       └── manila.conf
    │   ├── tests
    │   │   ├── basic_deployment.py
    │   │   ├── gate-basic-trusty-icehouse
    │   │   ├── gate-basic-trusty-liberty
    │   │   ├── gate-basic-trusty-mitaka
    │   │   ├── gate-basic-xenial-mitaka
    │   │   ├── README.md
    │   │   └── tests.yaml
    │   └── tox.ini
    ├── test-requirements.txt
    └── unit_tests
        ├── __init__.py
        ├── test_lib_new_manila_plugin_handlers.py
        └── test_new_manila_plugin_handlers.py

This is a reactive source charm layer that is used to build the final charm.
The files in the template are:

src/
  The source layer for the charm.  This layer is compiled with other layers to
  build the full charm.

src/config.yaml
  This is the config for the charm.  This will almost certainly need to be
  extended to support the configuration options that the backend provider needs
  to set in the config for ``manila.conf``.

src/layer.yaml
  The ``layer.yaml`` describes the reactive layers that are used to create the
  charm.  Unless the charm needs to relate to something else in addition to the
  manila charm, then this file doesn't need to be altered.

src/metadata.yaml
  The ``metadata.yaml`` file describes the relations to other charms.  This
  file won't need changing unless additional relations are needed for the charm
  (e.g. a connection to a master controller charm or something similar).

src/tox.ini
  The ``tox.ini`` file contains the configuration for ``tox`` to unittest, lint
  and build a charm. This file shouldn't need changing.

src/reactive/{package}_handlers.py
  This file contains the reactive handlers for the charm.  If the default
  behavior of the charm needs to be altered then this is the starting point
  for that change.

src/lib/charm/openstack/{package}.py
  This file contains the charm definition and logic to determine if the
  configuration is complete and to generate the configuration for the manila
  charm.  **This file will need editing**.

src/templates/{release}/manila.conf
  The template file makes it easier to write out the configuration section that
  will be supplied to the ``manila.conf`` file in the manila charm.  **This
  file will need editing**.  If the earliest release is something other than
  mitaka, then the folder name will need to be renamed to the earliest release.

src/tests/*
  These are the functional tests that can be run on the charm to demonstrate
  that it is functionally correct.  The version in the template checks that the
  configuration gets written.  However, it will probably have to be edited to
  provide sufficient config to the plugin charm so that it will not be blocked.

unit_tests/*
  The unit_tests files may be used to demonstrate that the functions used
  within the charm layer are correct, which is especially useful for reducing
  regression tests.  It is recommended that the unit_tests files are inspected
  and altered to be test any functionality that is included in the charm.


How the subordinate charm updates the manila.conf file
======================================================

The basic theory of operation of a manila configuration plugin charm is to use
the config that is presented to the charm to write a configuration section for
a backend for manila.  The charm may also need to install software, and the
charm can be altered to do this, but normally the manila software comes with
all of the supported drivers as part of the code base.  Here only the
configuration is considered.

The plugin charm has access to the same authentication credentials as the
manila charm if it needs to configure OpenStack services or needs to write
authentication credentials to other configuration files.  The manila-generic
charm needs to configure [nova], [neutron] and [glance] sections and uses the
authentication data to do so.

1. The charm author should first modify the
   ``src/templates/mitaka/manila.conf`` file which contains the section that
   will be used to configure the backend.

2. Then the charm author will modify the
   ``src/lib/charm/openstack/{pacakge}.py`` file.

Configuration Options
---------------------

It is probably that the configuration charm will use config parameters as part
of the template.  This are exposed via the ``config.`` option in the template,
and on the ``options`` member in the charm instance.  It is sometimes useful to
compute a configuration option that can be used in the template or charm (e.g.
a boolean to say that the config is available).

A computed config option is done as:

.. code:: python

    @charms_openstack.adapters.config_property
    def is_config_okay(config):
        if config.something and config.something_else > 10:
            return True
        return False

This can then be used as ``{{ config.is_config_okay}}`` in a template or in the
charm instance as:

.. code:: python

    def some_method(self):
        if self.options.is_config_okay:
            do_something_if_the_config_is_okay()
        else:
            do_something_else()


Generating the configuration section
------------------------------------

The configuration for ``manila.conf`` is generated in the
``get_config_for_principal()`` method in the charm class defined in the
``src/lib/charm/openstack`` directory.  The key steps to be aware of are:

1. If the configuration is not complete or can't be generated for the backend
   then the function should return an empty dictionary: {}

2. The default template only assumes that the ``manila-plugin.available`` state
   is required to render the config.  If your interfaces (via states) are
   needed then they should be added as appropriate.

PEP8 the charm
==============

It's useful to verify that the charm code is valid Python code and that all the
imports needed are met and other *linting* issues.  With ``tox`` installed,
this can easily be done by:

.. code:: bash

    tox -e pep8


Build Charm
===========

The charm now needs to be built to pull down all the interfaces and layers the
charm depends on and rolled into the built charm which can be deployed.

.. code:: bash

    tox -e build

Deploy/Test Charm
=================

Testing/deploying the charm can only be really done with a fragment of an
OpenStack system and the tox ``func27-smoke`` target gives an easy method to
deploy and verify the small system.  Note that once it is deployed, if there
are errors then they can be modified and the charm rebuilt and re-deployed.  A
test session might look something like:

.. code:: bash

    cd build/builds/{package-name}
    tox -e func27-smoke   # this will run the gate-basic-xenial-mitaka

This will get an OpenStack fragment running.  The gate-basic-xenial-mitaka may
need to be changed if that target is not supported  by the charm.


Then if there are errors:

.. code:: bash

    cd {package-name}
    # make changes
    tox -e build
    juju remove-relation manila {package-name}
    # wait until the subordinate is removed/and or destroy the manila unit.
    # If destroying the manila unit, then remember to redploy it
    juju upgrade-charm --path=build/builds/{package-name} {package-name}
    juju add-relation manila {package-name}

This will re-install the subordinate charm which may show further errors, etc.

``juju status`` will now show both charms deployed.
