===============================
Zed (Draft version in progress)
===============================

The Zed OpenStack Charms release includes updates for the charms described on
the :doc:`../project/openstack-charms` page. As of this release, the project
consists of 62 stable charms.

For the list of bugs resolved in this release refer to the `Zed milestone`_ in
Launchpad.

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
   performing the different upgrade types see :doc:`../admin/upgrades/overview`
   page.

.. contents:: Summary of changes:
   :local:
   :depth: 2
   :backlinks: top

New stable charms
-----------------

<TITLE>
~~~~~~~

New stable charm features
-------------------------

With each new feature, there is a corresponding example bundle in the form of a
test bundle, and/or a section in the :doc:`../index`, that details its usage.
Test bundles are located in the ``src/tests/bundles`` directory of the relevant
charm repository (see all `charm repositories`_).

<TITLE>
~~~~~~~

ceph-mon charm: COS Lite support
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Support for sending metrics to prometheus-k8s in the `COS Lite
observability stack`_ has been added.

ovn charms: default to OVN 22.03 on Focal
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

On Focal, deployments of the OVN charms when using track 22.03
(``--channel 22.03/stable``) will use a new OVN specific UCA pocket. This is to
ensure that a machine can simultaneously have OpenStack components from one
source and OVN components from another.

This has no effect when upgrading the OVN charms or when scaling an existing
application (new units). However, a new configuration option `ovn-source`_ has
been added that can override this behavior (i.e. payload upgrades will be
triggered by charm upgrades and scaling).

Documentation updates
---------------------

<TITLE>
~~~~~~~

New tech-preview charms
-----------------------

keystone-openidc
~~~~~~~~~~~~~~~~

The keystone-openidc charm provides OpenID Connect support for the OpenStack
Keystone service. It is a subordinate charm used in conjunction with the
keystone principal charm. When related to the openstack-dashboard charm it
provides Web SSO support for Horizon.

New tech-preview charm features
-------------------------------

<TITLE>
~~~~~~~

Informational notices
---------------------

NVIDIA vGPU Virtual Workstation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The Nova vGPU features in the Nova Compute charms where validated for use
as the graphical display driver for Virtual Workstation usage.

See the `Virtual GPU`_ documentation for details on how to configure vGPU
mediated device types for this specific use case.

Deprecation notices
-------------------


ceph-mon charm: prometheus machine charm
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Support for relating to the prometheus2 machine charm is deprecated
and will be removed at some point in the future.

See new charm feature `ceph-mon charm: COS Lite support`_ above.


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
.. _Zed milestone: https://launchpad.net/openstack-charms/+milestone/Zed
.. _charm repositories: https://opendev.org/openstack?sort=alphabetically&q=charm-&tab=
.. _COS Lite observability stack: https://charmhub.io/cos-lite
.. _ovn-source: https://charmhub.io/ovn-chassis/configure?channel=22.03/stable#ovn-source
.. _Virtual GPU: https://docs.openstack.org/charm-guide/latest/admin/vgpu.html

.. COMMITS

.. BUGS
