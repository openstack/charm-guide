=================================
All charms: migration to channels
=================================

All maintenance for stable charms occurs on the various explicitly-named
channels (i.e. not based on the ``latest`` track). These are the channels that
the charms must be migrated to.

.. important::

   In order to receive charm updates (e.g. bug fixes) it is highly recommended
   to migrate from legacy non-channel charms to charms that use a channel.

   See :doc:`../charm-delivery` for an overview of how OpenStack charms are
   distributed.

Background
----------

All charms are now served from the `Charmhub`_, regardless of which prefix
(``cs:`` or ``ch:``) is used to deploy charms.

Determine current versions
--------------------------

A charm's channel is selected based on its corresponding service's software
version. Use the information in the below two sub-sections to determine the
version running for each application in your deployment.

OpenStack service charms
~~~~~~~~~~~~~~~~~~~~~~~~

For OpenStack service charms, to get the running OpenStack version you can
inspect the value assigned to the ``openstack-origin`` charm configuration
option.

For example, if the :command:`juju config keystone openstack-origin` command
outputs 'focal-xena' then the running OpenStack version is Xena.

It is expected that all OpenStack service charms will report the same OpenStack
version.

All other charms
~~~~~~~~~~~~~~~~

For all other charms, utilise the :command:`juju status` command.

Examples:

* rabbitmq-server:

  .. code-block:: console

     App              Version  Status  Scale  Charm            Store       Channel  Rev  OS      Message
     rabbitmq-server  3.8.2    active      1  rabbitmq-server  charmstore  stable   117  ubuntu  Unit is ready

  RabbitMQ is running version ``3.8.2``.

* ceph-osd:

  .. code-block:: none

     App           Version  Status  Scale  Charm         Store       Channel  Rev  OS      Message
     ceph-osd      16.2.6   active      3  ceph-osd      charmstore  stable   315  ubuntu  Unit is ready (2 OSD)

  Ceph is running version ``16.2.6`` (Pacific).

  Since the Ceph channels are based on code names, as a convenience, a mapping
  of versions to code names is provided:

  +---------+-----------+
  | Version | Code name |
  +=========+===========+
  | 12.2.13 | Luminous  |
  +---------+-----------+
  | 13.2.9  | Mimic     |
  +---------+-----------+
  | 14.2.22 | Nautilus  |
  +---------+-----------+
  | 15.2.14 | Octopus   |
  +---------+-----------+
  | 16.2.6  | Pacific   |
  +---------+-----------+
  | 17.2.0  | Quincy    |
  +---------+-----------+

Select the channels
-------------------

The :doc:`../charm-delivery` page includes a list of all the tracks available
to the OpenStack charms.

Examples:

* if Ceph Octopus is running, then any Ceph charm that supports the
  ``octopus/stable`` channel should use that channel

* if OVN 20.03 is running, then any OVN charm that supports the
  ``20.03/stable`` channel should use that channel

* if RabbitMQ 3.8.2 is running, then the rabbitmq-server charm should use the
  ``3.8/stable`` channel

Based on this information, select the appropriate channel for each charm in
your deployment.

Upgrade Juju
------------

Upgrade every Juju component of the given deployment to Juju ``2.9``. This
includes the Juju client, the controller model, and the workload model. See the
`Juju upgrade documentation`_ for guidance.

Perform the migration
---------------------

The migration consists of replacing all charms with new but software-equivalent
charms. Technically, this is not an upgrade but a form of crossgrade.

.. note::

   There is no need to upgrade the current charms to their latest stable
   revision prior to the migration.

The charm of a currently-deployed application is migrated according to the
following syntax:

.. code-block:: none

   juju refresh --switch ch:<charm> --channel <channel> <application-name>

For example, if the selected channel for the rabbitmq-server charm is
``3.8/stable`` then:

.. code-block:: none

   juju refresh --switch ch:rabbitmq-server --channel 3.8/stable rabbitmq-server

The application argument represents the application as it appears in the model.
That is, it may be a named application (e.g. 'mysql' and not
'mysql-innodb-cluster').

Change operator behaviour
-------------------------

Once all of your deployment's charms have been migrated to channels it is
important to:

* stop using the ``cs:`` prefix when referencing charms, whether in bundles or
  on the command line. Use the ``ch:`` prefix instead. Note that Juju ``2.9``
  uses the ``ch:`` prefix by default on the command line.

* always specify a channel when deploying a charm (e.g. :command:`juju deploy
  --channel pacific/stable ceph-radosgw`)

.. LINKS
.. _Charmhub: https://charmhub.io
.. _Juju upgrade documentation: https://juju.is/docs/juju/upgrading
