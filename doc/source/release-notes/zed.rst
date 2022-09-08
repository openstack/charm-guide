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
.. _Upgrades overview: https://docs.openstack.org/project-deploy-guide/charm-deployment-guide/latest/upgrade-overview.html
.. _charm repositories: https://opendev.org/openstack?sort=alphabetically&q=charm-&tab=
.. _COS Lite observability stack: https://charmhub.io/cos-lite

.. COMMITS

.. BUGS
