========
Antelope
========

The Antelope OpenStack Charms release includes updates for the charms
described on the :doc:`../project/openstack-charms` page. As of this release,
the project consists of 62 stable charms.

For the list of bugs resolved in this release refer to the `Antelope
milestone`_ in Launchpad.

For scheduling information of past and future releases see the
:doc:`../project/release-schedule`.

.. note::

   Release notes contents is superseded by updated information published in the
   :doc:`index` (this guide) after the release of any given OpenStack Charms
   version.

.. important::

   Always upgrade to the latest stable charms before making any major changes
   to your cloud and before filing bug reports. Note that charm upgrades and
   OpenStack upgrades are functionally different. For instructions on
   performing the different upgrade types see the
   :doc:`../admin/upgrades/overview` page.

.. contents:: Summary of changes:
   :local:
   :depth: 2
   :backlinks: top

New stable charms
-----------------

ironic-dashboard
~~~~~~~~~~~~~~~~

This charm provides the `Ironic Dashboard plugin`_ for use with the `OpenStack
Dashboard charm`_

Usage example:

.. code:: none

  juju deploy --channel antelope/stable openstack-dashboard
  juju deploy --channel antelope/stable ironic-dashboard
  juju integrate openstack-dashboard:dashboard-plugin ironic-dashboard:dashboard


New stable charm features
-------------------------

With each new feature, there is a corresponding example bundle in the form of a
test bundle, and/or a section in the current guide (Charm Guide) that details
its usage. Test bundles are located in the ``src/tests/bundles`` directory of
the relevant charm repository (see all `charm repositories`_).

ironic-conductor charm: hardware enablement configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The ironic-conductor charm has acquired a new configuration option:

* ``hardware-enablement-options``

This option allows operators to include extra configuration options that
allows them to enable hardware specific options in the Ironic Conductor
service

For example to enable the `iDrac driver`_, the following commands can be used:

.. code-block:: none

   cat << EOF > ./idrac.ini
   [DEFAULT]
   enabled_hardware_types = intel-ipmi, ipmi, idrac
   enabled_management_interfaces = intel-ipmitool, ipmitool, noop, idrac-wsman
   enabled_inspect_interfaces = no-inspect, idrac-wsman
   enabled_power_interfaces = ipmitool, idrac-wsman
   enabled_console_interfaces = ipmitool-shellinabox, ipmitool-socat, no-console
   enabled_vendor_interfaces = ipmitool, no-vendor, idrac-wsman
   enabled_raid_interfaces = agent, no-raid, idrac-wsman
   EOF
   juju config ironic-conductor hardware-enablement-options=@./idrac.ini

Stable hostname for nova-compute service
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Provide a stable hostname for the nova-compute service when rendering the
``nova.conf`` file, this prevents the daemon from registering multiple entries
(with different hostnames) in the Nova control plane, also sticks to the same
hostname used by ovn-controller, this allows situations where a new instance
is allocated to nova-compute host "foo.example.com", but the ovn-chassis
registered is "foo", for more details see bug `LP #1896630`_.

ironic-conductor charm: Temporary url timeout
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The ironic-conductor charm has acquired a new configuration option:

* ``swift-temp-url-duration``

This option allows operators to fine tune the duration of temporary URLs
passed to ironic-python-agent to download the image that needs to be
installed, environments that use large images and/or slow IO baremetal nodes
are encouraged to increase it.

For example to set duration to one hour run:

.. code-block:: none

   juju config swift-temp-url-duration=3600

Documentation updates
---------------------

<TITLE>
~~~~~~~

New tech-preview charms
-----------------------

<TITLE>
~~~~~~~

New tech-preview charm features
-------------------------------

<TITLE>
~~~~~~~

Informational notices
---------------------

<TITLE>
~~~~~~~

Deprecation notices
-------------------

<TITLE>
~~~~~~~

Removed features
----------------

<TITLE>
~~~~~~~

Removed charms
--------------

<TITLE>
~~~~~~~

Issues discovered during this release cycle
-------------------------------------------

<TITLE>
~~~~~~~

.. LINKS
.. _Antelope milestone: https://launchpad.net/openstack-charms/+milestone/antelope
.. _Upgrades overview: https://docs.openstack.org/charm-guide/latest/admin/upgrades/overview.html
.. _charm repositories: https://opendev.org/openstack?sort=alphabetically&q=charm-&tab=
.. _Ironic Dashboard plugin: https://docs.openstack.org/ironic-ui/latest/
.. _OpenStack Dashboard charm: https://charmhub.io/openstack-dashboard
.. _iDrac driver: https://docs.openstack.org/ironic/latest/admin/drivers/idrac.html
.. COMMITS

.. BUGS
.. _LP #1896630: https://bugs.launchpad.net/charm-nova-compute/+bug/1896630
