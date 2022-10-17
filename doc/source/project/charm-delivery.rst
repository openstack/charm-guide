==============
Charm delivery
==============

Overview
--------

Juju charms are delivered via the online `Charmhub`_. This page provides charm
delivery information in the context of the OpenStack Charms project.

There are two general types of OpenStack charms: one that does use channels and
one that does not (legacy) - see the :doc:`../concepts/charm-types` for
details. This page will focus on channel charms.

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

   For deployments that are running non-channel (legacy) charms, please see
   special charm operation :doc:`procedures/charmhub-migration`.

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

The below tables give the tracks that are available for several categories of
charms. A mapping of tracks to both OpenStack release and Ubuntu series is also
provided.

.. note::

   The OpenStack service charms that support the following OpenStack releases
   will soon be managed by a single track: Queens, Rocky, Stein, and Train.

.. tabs::

   .. group-tab:: Charm groups

      +--------------------------+--------------+----------------+--------------+
      | Group                    | Tracks       | OpenStack      | Series       |
      +==========================+==============+================+==============+
      | OpenStack service charms | ``yoga``     | ``Yoga``       | | ``jammy``  |
      |                          |              |                | | ``focal``  |
      |                          +--------------+----------------+--------------+
      |                          | ``xena``     | ``Xena``       | ``focal``    |
      |                          +--------------+----------------+              |
      |                          | ``wallaby``  | ``Wallaby``    |              |
      |                          +--------------+----------------+              |
      |                          | ``victoria`` | ``Victoria``   |              |
      |                          +--------------+----------------+--------------+
      |                          | ``ussuri``   | ``Ussuri``     | | ``focal``  |
      |                          |              |                | | ``bionic`` |
      +--------------------------+--------------+----------------+--------------+
      | Ceph charms              | ``quincy``   | ``Yoga``       | | ``jammy``  |
      |                          |              |                | | ``focal``  |
      |                          +--------------+----------------+--------------+
      |                          | ``pacific``  | | ``Xena``     | ``focal``    |
      |                          |              | | ``Wallaby``  |              |
      |                          +--------------+----------------+              |
      |                          | ``octopus``  | ``Victoria``   |              |
      |                          |              +----------------+--------------+
      |                          |              | ``Ussuri``     | | ``focal``  |
      |                          |              |                | | ``bionic`` |
      |                          +--------------+----------------+--------------+
      |                          | ``nautilus`` | ``Train``      | ``bionic``   |
      |                          +--------------+----------------+              |
      |                          | ``mimic``    | | ``Stein``    |              |
      |                          |              | | ``Rocky``    |              |
      |                          +--------------+----------------+              |
      |                          | ``luminous`` | ``Queens``     |              |
      +--------------------------+--------------+----------------+--------------+
      | OVN charms               | ``22.03``    | ``Yoga``       | | ``jammy``  |
      |                          |              |                | | ``focal``  |
      |                          +--------------+----------------+--------------+
      |                          | ``21.09``    | ``Xena``       | ``focal``    |
      |                          +--------------+----------------+              |
      |                          | ``20.12``    | ``Wallaby``    |              |
      |                          +--------------+----------------+              |
      |                          | ``20.03``    | ``Victoria``   |              |
      |                          |              +----------------+--------------+
      |                          |              | ``Ussuri``     | | ``focal``  |
      |                          |              |                | | ``bionic`` |
      +--------------------------+--------------+----------------+--------------+
      | MySQL charms             | ``8.0``      | ``Yoga``       | | ``jammy``  |
      |                          |              |                | | ``focal``  |
      +--------------------------+--------------+----------------+--------------+

   .. group-tab:: Individual charms

      +--------------------------+--------------+----------------+--------------+
      | Charm                    | Tracks       | OpenStack      | Series       |
      +==========================+==============+================+==============+
      | hacluster                | ``2.4``      | ``Yoga``       | | ``jammy``  |
      |                          |              |                | | ``focal``  |
      |                          +--------------+----------------+--------------+
      |                          | ``2.0.3``    | | ``Xena``     | ``focal``    |
      |                          |              | | ``Wallaby``  |              |
      |                          |              | | ``Victoria`` |              |
      |                          |              +----------------+--------------+
      |                          |              | ``Ussuri``     | | ``focal``  |
      |                          |              |                | | ``bionic`` |
      |                          |              +----------------+--------------+
      |                          |              | | ``Train``    | ``bionic``   |
      |                          |              | | ``Stein``    |              |
      |                          |              | | ``Rocky``    |              |
      |                          |              | | ``Queens``   |              |
      +--------------------------+--------------+----------------+--------------+
      | openstack-loadbalancer   | ``jammy``    | ``Yoga``       | | ``jammy``  |
      |                          |              |                | | ``focal``  |
      +--------------------------+--------------+----------------+--------------+
      | pacemaker-remote         | ``2.4``      | ``Yoga``       | | ``jammy``  |
      |                          |              |                | | ``focal``  |
      |                          +--------------+----------------+--------------+
      |                          | ``2.0.3``    | | ``Xena``     | ``focal``    |
      |                          |              | | ``Wallaby``  |              |
      |                          |              | | ``Victoria`` |              |
      |                          |              +----------------+--------------+
      |                          |              | ``Ussuri``     | | ``focal``  |
      |                          |              |                | | ``bionic`` |
      |                          |              +----------------+--------------+
      |                          |              | | ``Train``    | ``bionic``   |
      |                          |              | | ``Stein``    |              |
      |                          |              | | ``Rocky``    |              |
      |                          |              | | ``Queens``   |              |
      +--------------------------+--------------+----------------+--------------+
      | rabbitmq-server          | ``3.9``      | ``Yoga``       | | ``jammy``  |
      |                          |              |                | | ``focal``  |
      |                          +--------------+----------------+--------------+
      |                          | ``3.8``      | | ``Xena``     | ``focal``    |
      |                          |              | | ``Wallaby``  |              |
      |                          |              | | ``Victoria`` |              |
      |                          |              +----------------+--------------+
      |                          |              | ``Ussuri``     | | ``focal``  |
      |                          |              |                | | ``bionic`` |
      |                          |              +----------------+--------------+
      |                          |              | | ``Train``    | ``bionic``   |
      |                          |              | | ``Stein``    |              |
      |                          |              | | ``Rocky``    |              |
      |                          |              | | ``Queens``   |              |
      +--------------------------+--------------+----------------+--------------+
      | vault                    | ``1.7``      | ``Yoga``       | | ``jammy``  |
      |                          |              |                | | ``focal``  |
      |                          |              +----------------+--------------+
      |                          |              | | ``Xena``     | ``focal``    |
      |                          |              | | ``Wallaby``  |              |
      |                          |              | | ``Victoria`` |              |
      |                          |              | | ``Ussuri``   |              |
      |                          +--------------+----------------+              |
      |                          | ``1.6``      | | ``Yoga``     |              |
      |                          |              | | ``Xena``     |              |
      |                          | and          | | ``Wallaby``  |              |
      |                          |              | | ``Victoria`` |              |
      |                          | ``1.5``      +----------------+--------------+
      |                          |              | ``Ussuri``     | | ``focal``  |
      |                          |              |                | | ``bionic`` |
      |                          |              +----------------+--------------+
      |                          |              | | ``Train``    | ``bionic``   |
      |                          |              | | ``Stein``    |              |
      |                          |              | | ``Rocky``    |              |
      |                          |              | | ``Queens``   |              |
      +--------------------------+--------------+----------------+--------------+
      | percona-cluster          | ``5.7``      | | ``Ussuri``   | ``bionic``   |
      |                          |              | | ``Train``    |              |
      |                          |              | | ``Stein``    |              |
      |                          |              | | ``Rocky``    |              |
      |                          |              | | ``Queens``   |              |
      +--------------------------+--------------+----------------+--------------+

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

Delivering a charm
------------------

A channel charm gets delivered to a deployment by using the ``--channel``
option with either the :command:`deploy` or :command:`refresh` commands. See
also the :doc:`../concepts/software-sources` page.

Deploying a charm
~~~~~~~~~~~~~~~~~

To deploy a channel charm select the channel that corresponds to the current
OpenStack release.

Examples,

To deploy the placement charm on an OpenStack Xena cloud the 'xena/stable'
channel is chosen:

.. code-block:: none

   juju deploy --channel xena/stable placement

To deploy the ceph-mon charm on an OpenStack Xena cloud the 'quincy/stable'
channel is chosen:

.. code-block:: none

   juju deploy --channel quincy/stable ceph-mon

.. _changing_the_channel:

Changing the channel
~~~~~~~~~~~~~~~~~~~~

A charm's channel is typically changed as part of an OpenStack upgrade. The
new channel must be chosen according to the target OpenStack release.

.. warning::

   Changing a charm's channel will trigger a charm upgrade, which will
   typically cause the underlying cloud service to restart.

   Study the :doc:`../admin/upgrades/openstack` process prior to changing charm
   channels.

Examples,

To change the channel for the vault charm when upgrading to OpenStack Yoga
the channel should be changed to 'yoga/stable':

.. code-block:: none

   juju refresh --channel 1.7/stable vault

To change the channel for the ovn-central charm when upgrading to OpenStack
Yoga the channel should be changed to '22.03/stable':

.. code-block:: none

   juju refresh --channel 22.03/stable ovn-central

.. LINKS
.. _Charmhub: https://charmhub.io
