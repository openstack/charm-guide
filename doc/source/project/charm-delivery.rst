==============
Charm delivery
==============

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
     - yoga, zed, 2023.1

   * - ceph-mon
     - pacific, quincy

   * - rabbitmq-server
     - 3.8, 3.9

   * - ovn-central
     - 22.09, 23.03

.. note::

   Track names for the collection of OpenStack service charms are based on the
   release names of the upstream OpenStack project. Notably, a rename occurred
   during the March 2023 release cycle where the expected upstream ``Antelope``
   release effectively became ``2023.1``. See the upstream `OpenStack
   Releases`_ page.

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

Tracks for the OpenStack Charms project
---------------------------------------

.. note::

   The information in this section will be updated as new tracks become
   available.

This section shows the **recommended** tracks for all the main charms that make
up the OpenStack Charms project. The information is categorised by the Ubuntu
release/series running on the cloud nodes.

.. important::

   Recall that the target OpenStack release is identified by the track
   associated with the OpenStack service charms (top row in each table).

A track can be specified during the deployment of an initial application or
during an OpenStack service upgrade. See the below section `Delivering a
charm`_ for details.

.. tabs::

   .. group-tab:: Ubuntu 22.04 LTS (Jammy)

      .. list-table::
         :header-rows: 1
         :widths: auto
         :stub-columns: 0

         * - Charms
           - Tracks
           -
           -

         * - OpenStack charms
           - ``yoga``
           - ``zed``
           - ``2023.1``

         * - Ceph charms
           - ``quincy``
           - ``quincy``
           - ``quincy``

         * - OVN charms
           - ``22.03``
           - ``22.09``
           - ``23.03``

         * - MySQL charms
           - ``8.0``
           - ``8.0``
           - ``8.0``

         * - hacluster
           - ``2.4``
           - ``2.4``
           - ``2.4``

         * - pacemaker-remote
           - ``jammy``
           - ``jammy``
           - ``jammy``

         * - rabbitmq-server
           - ``3.9``
           - ``3.9``
           - ``3.9``

         * - vault
           - ``1.8``
           - ``1.8``
           - ``1.8``

   .. group-tab:: Ubuntu 20.04 LTS (Focal)

      .. list-table::
         :header-rows: 1
         :widths: auto
         :stub-columns: 0

         * - Charms
           - Tracks
           -
           -
           -
           -

         * - OpenStack charms
           - ``ussuri``
           - ``victoria``
           - ``wallaby``
           - ``xena``
           - ``yoga``

         * - Ceph charms
           - ``octopus``
           - ``octopus``
           - ``pacific``
           - ``pacific``
           - ``quincy``

         * - OVN charms
           - ``22.03``
           - ``22.03``
           - ``22.03``
           - ``22.03``
           - ``22.03``

         * - MySQL charms
           - ``8.0``
           - ``8.0``
           - ``8.0``
           - ``8.0``
           - ``8.0``

         * - hacluster
           - ``2.0.3``
           - ``2.0.3``
           - ``2.0.3``
           - ``2.0.3``
           - ``2.0.3``

         * - pacemaker-remote
           - ``focal``
           - ``focal``
           - ``focal``
           - ``focal``
           - ``focal``

         * - rabbitmq-server
           - ``3.8``
           - ``3.8``
           - ``3.8``
           - ``3.8``
           - ``3.8``

         * - vault
           - ``1.7``
           - ``1.7``
           - ``1.7``
           - ``1.7``
           - ``1.7``

   .. group-tab:: Ubuntu 18.04 LTS (Bionic)

      .. list-table::
         :header-rows: 1
         :widths: auto
         :stub-columns: 0

         * - Charms
           - Tracks
           -
           -
           -
           -

         * - OpenStack charms
           - ``queens``
           - ``rocky``\ :sup:`EOL`
           - ``stein``\ :sup:`EOL`
           - ``train``\ :sup:`EOL`
           - ``ussuri``

         * - Ceph charms
           - ``luminous``
           - ``mimic``
           - ``mimic``
           - ``nautilus``
           - ``nautilus``

         * - pacemaker-remote
           - ``bionic``
           - ``bionic``
           - ``bionic``
           - ``bionic``
           - ``bionic``

         * - percona-cluster
           - ``5.7``
           - ``5.7``
           - ``5.7``
           - ``5.7``
           - ``5.7``

         * - hacluster
           - ``1.1.18``
           - ``1.1.18``
           - ``1.1.18``
           - ``1.1.18``
           - ``1.1.18``

         * - rabbitmq-server
           - ``3.6``
           - ``3.6``
           - ``3.6``
           - ``3.6``
           - ``3.6``

         * - vault
           - ``1.5``
           - ``1.5``
           - ``1.5``
           - ``1.5``
           - ``1.5``

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

A channel charm gets delivered by using the ``--channel`` option with either
the :command:`deploy` or :command:`refresh` commands. See also the
:doc:`../concepts/software-sources` page.

Deploying a charm
~~~~~~~~~~~~~~~~~

To deploy a channel charm select the channel that corresponds to the target
OpenStack release.

Examples,

To deploy the placement charm for an OpenStack Xena cloud the 'xena/stable'
channel is chosen:

.. code-block:: none

   juju deploy --channel xena/stable placement

To deploy the ceph-mon charm for an OpenStack Xena cloud the 'quincy/stable'
channel is chosen:

.. code-block:: none

   juju deploy --channel quincy/stable ceph-mon

.. _changing_the_channel:

Changing the channel
~~~~~~~~~~~~~~~~~~~~

A charm's channel is typically changed as part of an OpenStack upgrade. The
new channel must be chosen according to the target future OpenStack release.

.. warning::

   Changing a charm's channel is intended to trigger a charm upgrade, which
   will typically cause the underlying cloud service to restart.

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
.. _OpenStack Releases: https://releases.openstack.org
