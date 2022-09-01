================
Software sources
================

A key part of an OpenStack deploy or upgrade is the stipulation of an
application's software sources. This will affect the software used for cloud
services.

During an upgrade, the software source will naturally reflect a more recent
combination of Ubuntu release (series) and OpenStack release. This combination
is based on the `Ubuntu Cloud Archive`_ and translates to a "cloud archive
OpenStack release". It takes on the following syntax:

``<ubuntu series>-<openstack-release>``

The value is passed to a charm's ``openstack-origin`` configuration option. For
example, to select the 'focal-victoria' release:

``openstack-origin=cloud:focal-victoria``

In this way the charm can update the apt package sources for the machines that
correspond to a given application.

Notes concerning the value of ``openstack-origin``:

* The default is 'distro'. This denotes an Ubuntu release's default archive
  (e.g. in the case of the focal series it corresponds to OpenStack Ussuri).
  The value of 'distro' is therefore invalid in the context of an OpenStack
  upgrade.

* It should normally be the same across all charms.

* Its series component must be that of the series currently in use (i.e. a
  series upgrade and an OpenStack upgrade are two completely separate
  procedures).

.. note::

   A few charms use option ``source`` instead of ``openstack-origin`` (both
   options support identical values). The ``source`` option is used by charms
   that don't deploy an actual OpenStack service (e.g. ceph-osd).

.. LINKS
.. _Ubuntu Cloud Archive: https://wiki.ubuntu.com/OpenStack/CloudArchive
