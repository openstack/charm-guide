================
Software sources
================

A key part of an OpenStack deploy or upgrade is the stipulation of an
application's software sources. This will affect the software used for cloud
services.

This page references charm channels. For background information on channels see
the :doc:`../project/charm-delivery` page.

Values
------

A software source is typically expressed by the user with the following syntax:

``cloud:<ubuntu series>-<openstack-release>``

This value is based on the `Ubuntu Cloud Archive`_ and translates to a "cloud
archive OpenStack release".

For example: ``cloud:focal-xena``

Notes concerning the software source value:

* It should normally be the same across all charms.

* During an :doc:`../admin/upgrades/openstack`, it will reflect the next
  supported combination of Ubuntu release (series) and OpenStack release (e.g.
  'cloud:focal-xena' would become 'cloud:focal-yoga').

* The value of 'distro' denotes an Ubuntu release's default archive
  (e.g. in the case of the focal series it corresponds to OpenStack Ussuri).
  It is the default value.

* See the ``config.yaml`` file of the `keystone charm`_ for an example
  explanation of how to specify more exotic values (PPA, deb sources entry,
  alternate UCA pocket).

When using channels
~~~~~~~~~~~~~~~~~~~

In addition to the above traditional syntax, when channels are in use, a
software sources value can refer simply to an OpenStack release name.

For example: ``ussuri``

With this method, the value always represents the desired OpenStack release,
regardless of the underlying series. Notably, the case of an OpenStack release
spanning multiple series is supported. That is, a value of 'ussuri' works with
both Bionic (instead of 'cloud:bionic-ussuri') and Focal (rather than
'distro').

Configuration
-------------

Configuration is achieved by passing the value to charms via one of the
following charm configuration options:

* ``openstack-origin`` (for OpenStack service charms such as keystone or
  cinder)

* ``source`` (for non-OpenStack service charms such as ceph-osd or ovn-central)

In this way, apt package sources can be updated on all machines associated with
a given application.

The configuration should be set during deployment. For example, to deploy
Cinder on Focal for an OpenStack Xena cloud:

.. code-block:: none

   juju deploy --config openstack-origin=cloud:focal-xena cinder

To query the cinder charm for its software sources:

.. code-block:: none

   juju config cinder openstack-origin

When using channels
~~~~~~~~~~~~~~~~~~~

When a charm is delivered via a channel, the software sources will be
configured automatically. As expected, the case of an OpenStack release
spanning multiple series is supported. That is, ``--channel yoga/stable`` will
set the sources to 'yoga' on either Focal or Jammy:

.. code-block:: none

   juju deploy --channel yoga/stable cinder

Manual configuration should therefore only be needed when deploying via an
exotic source (covered earlier). For example, the above command automatically
performs:

.. code-block:: none

   juju config cinder openstack-origin=yoga

.. important::

   With channels, an OpenStack upgrade is not performed by changing the
   software sources manually. It involves :ref:`changing the charm's channel
   <changing_the_channel>`.

.. LINKS
.. _Ubuntu Cloud Archive: https://wiki.ubuntu.com/OpenStack/CloudArchive
.. _keystone charm: https://charmhub.io/keystone/configure#openstack-origin
