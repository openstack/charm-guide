=====================================================
Issues, charm procedures, and OpenStack upgrade notes
=====================================================

This page centralises all software issues, special procedures, and OpenStack
upgrade path notes.

.. important::

   The documentation of issues and procedures are not duplicated amid the
   below-mentioned pages. For instance, an issue related to a specific
   OpenStack upgrade path is not repeated in the
   :ref:`upgrade_issues_openstack_upgrades` section of the Upgrade issues page
   (an upgrade issue can potentially affect multiple OpenStack upgrade paths).

Software issues
---------------

Software issues mentioned here can be related to the charms or upstream
OpenStack. Both upgrade and various non-upgrade (but noteworthy) issues are
documented. Note that an upgrade issue can affect either of the three types of
upgrades (charms, OpenStack, series). The following pages document the most
important unresolved software issues in Charmed OpenStack:

.. toctree::
   :maxdepth: 1

   issues/upgrade-issues
   issues/various-issues

Special charm procedures
------------------------

During the lifetime of a cloud, the evolution of the upstream software or that
of the charms will necessitate action on the part of the cloud administrator.
This often involves replacing existing charms with new charms. For example, a
charm may become deprecated and be superseded by a new charm, resulting in a
recommendation (or a requirement) to use the new charm. The following is the
list of documented special charm procedures:

.. toctree::
   :maxdepth: 1

   procedures/ceph-charm-migration
   procedures/percona-series-upgrade-to-focal
   procedures/placement-charm
   procedures/cinder-lvm-migration
   procedures/charmhub-migration
   procedures/ovn-migration
   procedures/ovn-upgrade-2203

OpenStack upgrade path notes
----------------------------

Occasionally, a specific OpenStack upgrade path will require particular
attention. These upgrade notes typically contain software issues. If a special
procedure is pertinent however, it will be mentioned. It is important to study
the page that may apply to your upgrade scenario. The following is the list of
documented OpenStack upgrade path notes:

.. toctree::
   :maxdepth: 1

   issues/upgrade-2023.1-to-2023.2
   issues/upgrade-xena-to-yoga
   issues/upgrade-wallaby-to-xena
   issues/upgrade-victoria-to-wallaby
   issues/upgrade-ussuri-to-victoria
   issues/upgrade-stein-to-train
   issues/upgrade-queens-to-rocky
   issues/upgrade-newton-to-ocata
   issues/upgrade-mitaka-to-newton

.. LINKS
