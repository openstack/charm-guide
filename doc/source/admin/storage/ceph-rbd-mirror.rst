==================
Ceph RBD mirroring
==================

.. important::

   This page has been identified as being affected by the breaking changes
   introduced between versions 2.9.x and 3.x of the Juju client. Read
   support note :ref:`juju_29_3x_changes` before continuing.

Overview
--------

RADOS Block Device (RBD) mirroring is a process of asynchronous replication of
Ceph block device images between two or more Ceph clusters. Mirroring ensures
point-in-time consistent replicas of all changes to an image, including reads
and writes, block device resizing, snapshots, clones, and flattening. RBD
mirroring is mainly used for disaster recovery (i.e. having a secondary site as
a failover). See the Ceph documentation on `RBD mirroring`_ for complete
information.

This guide will show how to deploy two Ceph clusters with RBD mirroring between
them with the use of the ceph-rbd-mirror charm. See the `charm's
documentation`_ for basic information and charm limitations.

RBD mirroring is only one aspect of datacentre redundancy. Refer to
:doc:`ceph-rgw-multisite` and other work to arrive at a complete solution.

Performance considerations
--------------------------

RBD mirroring makes use of the journaling feature of Ceph. This incurs an
overhead for write activity on an RBD image that will adversely affect
performance.

For more information on performance aspects see Florian Haas' talk
`Geographical Redundancy with rbd-mirror`_ (video) given at Cephalocon
Barcelona 2019.

Requirements
------------

The two Ceph clusters will correspond to sites 'a' and 'b' and each cluster
will reside within a separate model (models 'site-a' and 'site-b'). The
deployment will require the use of `Cross model relations`_.

Deployment characteristics:

* each cluster will have 7 units:

  * 3 x ceph-osd
  * 3 x ceph-mon
  * 1 x ceph-rbd-mirror

* application names will be used to distinguish between the applications in
  each site (e.g. site-a-ceph-mon and site-b-ceph-mon)

* the ceph-osd units will use block device ``/dev/vdd`` for their OSD volumes

.. note::

   The two Ceph clusters can optionally be placed within the same model, and
   thus obviate the need for cross model relations. This topology is not
   generally considered to be a real world scenario.

Deployment
----------

For site 'a' the following configuration is placed into file ``site-a.yaml``:

.. code-block:: yaml

   site-a-ceph-mon:
     monitor-count: 3
     expected-osd-count: 3
     source: distro

   site-a-ceph-osd:
     osd-devices: /dev/vdd
     source: distro

   site-a-ceph-rbd-mirror:
     source: distro

Create the model and deploy the software for each site:

* Site 'a'

  .. code-block:: none

     juju add-model site-a
     juju deploy -n 3 --config site-a.yaml ceph-osd site-a-ceph-osd
     juju deploy -n 3 --config site-a.yaml ceph-mon site-a-ceph-mon
     juju deploy --config site-a.yaml ceph-rbd-mirror site-a-ceph-rbd-mirror

* Site 'b'

  An analogous configuration file is used (i.e. replace 'a' with 'b'):

  .. code-block:: none

     juju add-model site-b
     juju deploy -n 3 --config site-b.yaml ceph-osd site-b-ceph-osd
     juju deploy -n 3 --config site-b.yaml ceph-mon site-b-ceph-mon
     juju deploy --config site-b.yaml ceph-rbd-mirror site-b-ceph-rbd-mirror

Add two local relations for each site:

* Site 'a'

  .. code-block:: none

     juju integrate -m site-a site-a-ceph-mon:osd site-a-ceph-osd:mon
     juju integrate -m site-a site-a-ceph-mon:rbd-mirror site-a-ceph-rbd-mirror:ceph-local

* Site 'b'

  .. code-block:: none

     juju integrate -m site-b site-b-ceph-mon:osd site-b-ceph-osd:mon
     juju integrate -m site-b site-b-ceph-mon:rbd-mirror site-b-ceph-rbd-mirror:ceph-local

Export a ceph-rbd-mirror endpoint (by means of an "offer") for each site. This
will enable us to create the inter-site (cross model) relations:

* Site 'a'

  .. code-block:: none

     juju switch site-a
     juju offer site-a-ceph-rbd-mirror:ceph-remote

  Output:

  .. code-block:: console

     Application "site-a-ceph-rbd-mirror" endpoints [ceph-remote] available at "admin/site-a.site-a-ceph-rbd-mirror"

* Site 'b'

  .. code-block:: none

     juju switch site-b
     juju offer site-b-ceph-rbd-mirror:ceph-remote

  Output:

  .. code-block:: console

     Application "site-b-ceph-rbd-mirror" endpoints [ceph-remote] available at "admin/site-b.site-b-ceph-rbd-mirror"

Add the two inter-site relations by referring to the offer URLs (included in
the output above) as if they were applications in the local model:

.. code-block:: none

   juju integrate -m site-a site-a-ceph-mon admin/site-b.site-b-ceph-rbd-mirror
   juju integrate -m site-b site-b-ceph-mon admin/site-a.site-a-ceph-rbd-mirror

Verify the output of :command:`juju status` for each model:

.. code-block:: none

   juju status -m site-a --relations

Output:

.. code-block:: console

   Model   Controller  Cloud/Region       Version  SLA          Timestamp
   site-a  prod-1      openstack/default  2.8.9    unsupported  16:00:39Z

   SAAS                    Status   Store        URL
   site-b-ceph-rbd-mirror  waiting  serverstack  admin/site-b.site-b-ceph-rbd-mirror

   App                     Version  Status   Scale  Charm            Store       Rev  OS      Notes
   site-a-ceph-mon         15.2.8   active       3  ceph-mon         jujucharms   53  ubuntu
   site-a-ceph-osd         15.2.8   active       3  ceph-osd         jujucharms  308  ubuntu
   site-a-ceph-rbd-mirror  15.2.8   waiting      1  ceph-rbd-mirror  jujucharms   15  ubuntu

   Unit                       Workload  Agent  Machine  Public address  Ports  Message
   site-a-ceph-mon/0          active    idle   0        10.5.0.4               Unit is ready and clustered
   site-a-ceph-mon/1          active    idle   1        10.5.0.14              Unit is ready and clustered
   site-a-ceph-mon/2*         active    idle   2        10.5.0.7               Unit is ready and clustered
   site-a-ceph-osd/0          active    idle   0        10.5.0.4               Unit is ready (1 OSD)
   site-a-ceph-osd/1          active    idle   1        10.5.0.14              Unit is ready (1 OSD)
   site-a-ceph-osd/2*         active    idle   2        10.5.0.7               Unit is ready (1 OSD)
   site-a-ceph-rbd-mirror/0*  waiting   idle   3        10.5.0.11              Waiting for pools to be created

   Machine  State    DNS        Inst id                               Series  AZ    Message
   0        started  10.5.0.4   4f3e4d94-5003-4998-ab30-11fc3c845a7a  focal   nova  ACTIVE
   1        started  10.5.0.14  7682822e-4469-41e1-b938-225c067f9f82  focal   nova  ACTIVE
   2        started  10.5.0.7   786e7d84-3f94-4cd6-9493-72026d629fcf  focal   nova  ACTIVE
   3        started  10.5.0.11  715c8738-e41e-4be2-8638-560206b2c434  focal   nova  ACTIVE

   Offer                   Application             Charm            Rev  Connected  Endpoint     Interface        Role
   site-a-ceph-rbd-mirror  site-a-ceph-rbd-mirror  ceph-rbd-mirror  15   1/1        ceph-remote  ceph-rbd-mirror  requirer

   Relation provider           Requirer                            Interface        Type     Message
   site-a-ceph-mon:mon         site-a-ceph-mon:mon                 ceph             peer
   site-a-ceph-mon:osd         site-a-ceph-osd:mon                 ceph-osd         regular
   site-a-ceph-mon:rbd-mirror  site-a-ceph-rbd-mirror:ceph-local   ceph-rbd-mirror  regular
   site-a-ceph-mon:rbd-mirror  site-b-ceph-rbd-mirror:ceph-remote  ceph-rbd-mirror  regular

   Model   Controller   Cloud/Region    Version  SLA          Timestamp
   site-a  maas-prod-1  acme-1/default  2.8.1    unsupported  20:00:41Z

.. code-block:: none

   juju status -m site-b --relations

Output:

.. code-block:: console

   Model   Controller  Cloud/Region       Version  SLA          Timestamp
   site-b  prod-1      openstack/default  2.8.9    unsupported  16:05:39Z

   SAAS                    Status   Store        URL
   site-a-ceph-rbd-mirror  waiting  serverstack  admin/site-a.site-a-ceph-rbd-mirror

   App                     Version  Status   Scale  Charm            Store       Rev  OS      Notes
   site-b-ceph-mon         15.2.8   active       3  ceph-mon         jujucharms   53  ubuntu
   site-b-ceph-osd         15.2.8   active       3  ceph-osd         jujucharms  308  ubuntu
   site-b-ceph-rbd-mirror  15.2.8   waiting      1  ceph-rbd-mirror  jujucharms   15  ubuntu

   Unit                       Workload  Agent  Machine  Public address  Ports  Message
   site-b-ceph-mon/0          active    idle   0        10.5.0.3               Unit is ready and clustered
   site-b-ceph-mon/1          active    idle   1        10.5.0.20              Unit is ready and clustered
   site-b-ceph-mon/2*         active    idle   2        10.5.0.8               Unit is ready and clustered
   site-b-ceph-osd/0          active    idle   0        10.5.0.3               Unit is ready (1 OSD)
   site-b-ceph-osd/1          active    idle   1        10.5.0.20              Unit is ready (1 OSD)
   site-b-ceph-osd/2*         active    idle   2        10.5.0.8               Unit is ready (1 OSD)
   site-b-ceph-rbd-mirror/0*  waiting   idle   3        10.5.0.12              Waiting for pools to be created

   Machine  State    DNS        Inst id                               Series  AZ    Message
   0        started  10.5.0.3   2caf61f7-8675-4cd9-a3c4-cc68a0cb3f2d  focal   nova  ACTIVE
   1        started  10.5.0.20  d1b3bd0b-1631-4bd3-abba-14a366b3d752  focal   nova  ACTIVE
   2        started  10.5.0.8   84eb5db2-d673-4d36-82b4-902463362704  focal   nova  ACTIVE
   3        started  10.5.0.12  c40e1247-7b7d-4b84-ab3a-8b72c22f096e  focal   nova  ACTIVE

   Offer                   Application             Charm            Rev  Connected  Endpoint     Interface        Role
   site-b-ceph-rbd-mirror  site-b-ceph-rbd-mirror  ceph-rbd-mirror  15   1/1        ceph-remote  ceph-rbd-mirror  requirer

   Relation provider           Requirer                            Interface        Type     Message
   site-b-ceph-mon:mon         site-b-ceph-mon:mon                 ceph             peer
   site-b-ceph-mon:osd         site-b-ceph-osd:mon                 ceph-osd         regular
   site-b-ceph-mon:rbd-mirror  site-a-ceph-rbd-mirror:ceph-remote  ceph-rbd-mirror  regular
   site-b-ceph-mon:rbd-mirror  site-b-ceph-rbd-mirror:ceph-local   ceph-rbd-mirror  regular

There are no Ceph pools created by default. The next section ('Pool creation')
provides guidance.

Pool creation
-------------

RBD pools can be created by either a supporting charm (through the Ceph broker
protocol) or manually by the operator:

#. A charm-created pool (e.g. the glance or nova-compute charms) will
   automatically be detected and acted upon (i.e. a remote pool will be set up
   in the peer cluster).

#. A manually-created pool, whether done via the ceph-mon application or
   through Ceph directly, will require an action to be run on the
   ceph-rbd-mirror application leader in order for the remote pool to come
   online.

   For example, to create a pool manually in site 'a' and have ceph-rbd-mirror
   (of site 'a') initialise a pool in site 'b':

   .. code-block:: none

      juju run --wait -m site-a site-a-ceph-mon/leader create-pool name=mypool app-name=rbd
      juju run --wait -m site-a site-a-ceph-rbd-mirror/leader refresh-pools

   This can be verified by listing the pools in site 'b':

   .. code-block:: none

      juju run --wait -m site-b site-b-ceph-mon/leader list-pools

.. note::

   Automatic peer-pool creation (for a charm-created pool) is based on the
   local pool being labelled with a Ceph 'rbd' tag. This Ceph-internal
   labelling occurs when the newly-created local pool is associated with the
   RBD application. This last feature is supported starting with Ceph Luminous
   (OpenStack Queens).

Failover and fallback
---------------------

To manage failover and fallback, the ``demote`` and ``promote`` actions are
applied to the ceph-rbd-mirror application leader.

For instance, to fail over from site 'a' to site 'b' the former is demoted and
the latter is promoted. The rest of the commands are status checks:

.. code-block:: none

   juju run --wait -m site-a site-a-ceph-rbd-mirror/leader status verbose=true
   juju run --wait -m site-b site-b-ceph-rbd-mirror/leader status verbose=true

   juju run --wait -m site-a site-a-ceph-rbd-mirror/leader demote

   juju run --wait -m site-a site-a-ceph-rbd-mirror/leader status verbose=true
   juju run --wait -m site-b site-b-ceph-rbd-mirror/leader status verbose=true

   juju run --wait -m site-b site-b-ceph-rbd-mirror/leader promote

To fall back to site 'a' the actions are reversed:

.. code-block:: none

   juju run --wait -m site-b site-b-ceph-rbd-mirror/leader demote
   juju run --wait -m site-a site-a-ceph-rbd-mirror/leader promote

.. note::

   With Ceph Luminous (and greater), the mirror status information may not be
   accurate. Specifically, the ``entries_behind_master`` counter may never get
   to '0' even though the image has been fully synchronised.

Recovering from abrupt shutdown
-------------------------------

It is possible that an abrupt shutdown and/or an interruption to communication
channels may lead to a "split-brain" condition. This may cause the mirroring
daemon in each cluster to claim to be the primary. In such cases, the operator
must make a call as to which daemon is correct. Generally speaking, this means
deciding which cluster has the most recent data.

Elect a primary by applying the ``demote`` and ``promote`` actions to the
appropriate ceph-rbd-mirror leader. After doing so, the ``resync-pools`` action
must be run on the secondary cluster leader. The ``promote`` action may require
a force option.

Here, we make site 'a' be the primary by demoting site 'b' and promoting site
'a':

.. code-block:: none

   juju run --wait -m site-b site-b-ceph-rbd-mirror/leader demote
   juju run --wait -m site-a site-a-ceph-rbd-mirror/leader promote force=true

   juju run --wait -m site-a site-a-ceph-rbd-mirror/leader status verbose=true
   juju run --wait -m site-b site-b-ceph-rbd-mirror/leader status verbose=true

   juju run --wait -m site-b site-b-ceph-rbd-mirror/leader resync-pools i-really-mean-it=true

.. note::

   When using Ceph Luminous, the mirror state information will not be accurate
   after recovering from unclean shutdown. Regardless of the output of the
   status information, you will be able to write to images after a forced
   promote.

.. LINKS
.. _charm's documentation: https://opendev.org/openstack/charm-ceph-rbd-mirror/src/branch/master/src/README.md
.. _RBD mirroring: https://docs.ceph.com/en/latest/rbd/rbd-mirroring
.. _Geographical Redundancy with rbd-mirror: https://youtu.be/ZifNGprBUTA
.. _Cross model relations: https://juju.is/docs/juju/cross-model-integration
