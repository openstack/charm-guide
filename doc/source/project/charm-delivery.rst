==============
Charm delivery
==============

Overview
--------

Juju charms are delivered via the online `Charmhub`_. This page provides charm
delivery information in the context of the OpenStack Charms project.

Channels
--------

The notion of channels is central to the delivery of the correct OpenStack
charm.

By definition, a channel is comprised of three components: track, risk level,
and branch. If a component is not specified at deploy time a default value will
be used.

Unlike many charms in the Charmhub, the OpenStack Charms project leverages
tracks extensively in order to manage a different array of supported charm
versions for each supported combination of OpenStack release and Ubuntu series.

The Charmhub can be queried for a charm's supported channels. For utmost
clarity, this should be done on a per-series basis. For example, to query the
channels for the glance charm on the 'focal' series:

.. code-block:: none

   juju info --series focal glance

Alternatively, the Charmhub web site can be visited: https://charmhub.io/glance

.. important::

   For deployments that are still running legacy non-channel charms, please
   consult special charm operation "All charms: migrating to channels".

Tracks
~~~~~~

A charm's track corresponds to a given release of its payload. When the
associated upstream project uses well-known release names, the track will use
that name as its version number. Otherwise, the track's name will typically be
based on a version number or a release date. For example:

.. list-table::
   :header-rows: 1

   * - Charm
     - Example tracks

   * - nova-compute
     - wallaby, xena

   * - ceph-mon
     - octopus, pacific

   * - rabbitmq-server
     - 3.8, 3.9

   * - ovn-central
     - 21.03, 21.09

``latest`` track
^^^^^^^^^^^^^^^^

Generally speaking, the ``latest`` track is a special track whose purpose is to
conveniently point to the latest version of a piece of software.

.. warning::

   The OpenStack Charms project strongly advises against the use of the
   ``latest`` track due to its implicit nature. In doing so, a future charm
   upgrade may result in a charm version that does not support your current
   OpenStack release.

Risk levels
~~~~~~~~~~~

There are four risk levels that are potentially available for any track:
``edge``, ``beta``, ``candidate``, and ``stable``. Charms will be promoted
through these levels based on the OpenStack Charms team policy on software
maturity.

Branches
~~~~~~~~

There is the possibility for a further subdivision of tracks and risk levels
into Charmhub "branches", which are meant to provide short-lived releases (e.g.
quick bug fix).

Channels and tracks for OpenStack charms
----------------------------------------

.. important::

   The information in this section will be updated as new tracks become
   available.

.. note::

   Charms that support the following OpenStack releases will soon be managed by
   a single track: Queens, Rocky, Stein, and Train.

The below tables give the tracks that are available for several categories of
charms across all series.

.. tabs::

   .. group-tab:: Charm groups

      +--------------------------+--------------+
      | Group                    | Tracks       |
      +==========================+==============+
      | OpenStack service charms | ``yoga``     |
      |                          +--------------+
      |                          | ``xena``     |
      |                          +--------------+
      |                          | ``wallaby``  |
      |                          +--------------+
      |                          | ``victoria`` |
      |                          +--------------+
      |                          | ``ussuri``   |
      +--------------------------+--------------+
      | Ceph charms              | ``quincy``   |
      |                          +--------------+
      |                          | ``pacific``  |
      |                          +--------------+
      |                          | ``octopus``  |
      +--------------------------+--------------+
      | OVN charms               | ``22.03``    |
      |                          +--------------+
      |                          | ``21.09``    |
      |                          +--------------+
      |                          | ``20.12``    |
      |                          +--------------+
      |                          | ``20.03``    |
      +--------------------------+--------------+
      | MySQL charms             | ``8.0``      |
      +--------------------------+--------------+
      | Trilio charms            | ``4.1``      |
      |                          +--------------+
      |                          | ``4.0``      |
      +--------------------------+--------------+

   .. group-tab:: Supporting charms

      +--------------------------+--------------+
      | Charm                    | Tracks       |
      +==========================+==============+
      | hacluster                | ``2.0.5``    |
      +                          +--------------+
      |                          | ``2.0.3``    |
      +                          +--------------+
      |                          | ``1.1.x``    |
      +--------------------------+--------------+
      | pacemaker-remote         | ``2.0.5``    |
      +                          +--------------+
      |                          | ``2.0.3``    |
      +                          +--------------+
      |                          | ``1.1.x``    |
      +--------------------------+--------------+
      | rabbitmq-server          | ``3.9``      |
      +                          +--------------+
      |                          | ``3.8``      |
      +                          +--------------+
      |                          | ``3.6``      |
      +--------------------------+--------------+
      | vault                    | ``1.7``      |
      +                          +--------------+
      |                          | ``1.6``      |
      +                          +--------------+
      |                          | ``1.5``      |
      +--------------------------+--------------+
      | percona-cluster          | ``5.7``      |
      +--------------------------+--------------+

Provider-specific subordinate charms
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Some services interact with provider-specific subordinate charms in order to
enable a specific SDN, storage plugin, etc. Although these are considered
supporting charms, they nonetheless often enable specific functionality for an
OpenStack service. They therefore follow the same track-naming schema as do the
OpenStack service charms.

This is the list of provider-specific subordinate charms:

* cinder-ceph
* cinder-lvm
* cinder-netapp
* cinder-purestorage
* neutron-openvswitch
* neutron-api-plugin-arista
* neutron-api-plugin-ironic
* neutron-api-plugin-ovn
* keystone-saml-mellon

Installation sources
--------------------

Most charms in the OpenStack Charm project support either the
``openstack-origin`` or ``source`` configuration option. This options sets the
software sources of the hosting machine.

In order to ensure that a charm's channel will lead to the installation of the
correct software version, these options will be set automatically according to
the associated track. This is particularly important when a track spans
multiple series (e.g. Ussuri is supported on both the 'bionic' and 'focal'
series).

.. LINKS
.. _Charmhub: https://charmhub.io
