.. _release_notes_<CHARM_RELEASE>:

* * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * *
*                                                             *
* This is a template to be used for future release notes.     *
* Make a copy, remove this notice, and use the placeholders.  *
*                                                             *
* * * * * * * * * * * * * * * * * * * * * * * * * * * * * * * *

===========================================
<CHARM_RELEASE> (Draft version in progress)
===========================================

Summary of changes
------------------

* New charm features
   * `<TITLE>`_

* New tech-preview charms
   * `<TITLE>`_

* Deprecation notices
   * `<TITLE>`_

* Removed features
   * `<TITLE>`_

Overview
--------

The <CHARM_RELEASE> OpenStack Charms release includes updates for the charms
described on the `Charms`_ page. As of this release, the project consists of
<NUMBER-OF-STABLE-CHARMS> supported (stable) charms.

For the list of bugs resolved in this release refer to the `<CHARM_RELEASE>
milestone`_ in Launchpad.

For scheduling information of past and future releases see the `Release
schedule`_.

General charm information is published in the `OpenStack Charm Guide`_ (this
guide) which ultimately supersedes Release Notes contents.

.. important::

   Always upgrade to the latest stable charms before making any major changes
   to your cloud and before filing bug reports. Refer to section `Upgrading
   charms`_ below for details.

New charm features
------------------

With each new feature, there is a corresponding example bundle in the form of a
test bundle, and/or a section in the `OpenStack Charms Deployment Guide`_, that
details its usage. Test bundles are located in the ``src/tests/bundles``
directory of the relevant charm repository (see all `charm repositories`_).

<NEW-FEATURE-TITLE>
~~~~~~~~~~~~~~~~~~~

.. COMMENT
   New stable charms
   -----------------

.. COMMENT
   New tech-preview charms
   -----------------------

.. COMMENT
   Preview charm features
   ----------------------

.. COMMENT
   Documentation updates
   ---------------------

.. COMMENT
   Informational notices
   ---------------------

.. COMMENT
   Deprecation notices
   -------------------

.. COMMENT
   Removed features
   ----------------

.. COMMENT
   Removed charms
   --------------

.. COMMENT
   Issues discovered during this release cycle
   -------------------------------------------

Upgrading charms
----------------

Upgrading charms will making available new features and bug fixes. However, the
latest stable charm revision should also be used prior to making any
topological changes, application migrations, workload upgrades, or series
upgrades. Bug reports should also be filed against the most recent revision.

Note that charm upgrades and OpenStack upgrades are functionally different. For
instructions on performing the different upgrade types see `Upgrades overview`_
in the `OpenStack Charms Deployment Guide`_.

.. LINKS
.. _Charms: openstack-charms.html
.. _<CHARM_RELEASE> milestone: https://launchpad.net/openstack-charms/+milestone/<CHARM_RELEASE>
.. _OpenStack Charms Deployment Guide: https://docs.openstack.org/project-deploy-guide/charm-deployment-guide/latest
.. _OpenStack Charm Guide: https://docs.openstack.org/charm-guide/latest/
.. _Release schedule: release-schedule.html
.. _Upgrades overview: https://docs.openstack.org/project-deploy-guide/charm-deployment-guide/latest/upgrade-overview.html
.. _charm repositories: https://opendev.org/openstack?sort=alphabetically&q=charm-&tab=

.. COMMITS

.. BUGS
