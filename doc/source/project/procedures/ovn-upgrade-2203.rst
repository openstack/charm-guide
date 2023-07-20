=============================
Upgrade OVN to 22.03 on Focal
=============================

Charmed OpenStack supports OVN version 22.03 starting with OpenStack Ussuri.
Clouds running on Focal nodes that are not using this version are strongly
recommended to upgrade to it in order to benefit from important bug fixes and
software enhancements.

In particular, the procedure described on this page aims to prevent OVN data
plane downtime during the upgrade to 22.03. This is an `upstream OVN issue`_
that can cause network disruption to all cloud VMs.

.. important::

   Read this entire document before making any changes to your cloud.

   It is recommended that the upgrade be tested in a staged environment prior
   to applying the steps to a production cloud.

Prerequisites
-------------

Ensure the following prerequisites are satisfied before making any changes.

Juju version
~~~~~~~~~~~~

Juju must be running the latest stable version of its major and minor release
(e.g. 2.9.x). This pertains to all three Juju components: client, controller
model, and workload model. `Juju upgrade documentation`_ is available, but
quick guidance is also given below.

First ensure that the client context is the cloud's controller and model (check
with command :command:`juju whoami`). The essential commands are then:

.. code-block:: none

   sudo snap refresh juju
   juju upgrade-controller
   juju upgrade-model

Channel charms
~~~~~~~~~~~~~~

OVN must be managed by :doc:`channel charms <../../concepts/charm-types>`
(charms ovn-central and ovn-chassis). If it is not, perform the migration away
from legacy charms by applying special procedure :doc:`charmhub-migration` to
those two charms.

.. caution::

   As the above migration document states, when performing the migration to
   channel charms, ensure that the currently running OVN version does not
   change.

Updated neutron-api-plugin-ovn charm
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The neutron-api-plugin-ovn charm must support the ``ovn-source`` configuration
option. This works for the ``xena``, ``wallaby``, ``victoria``, and ``ussuri``
tracks.

To verify whether the option is available, query for its value:

.. code-block:: none

   juju config neutron-api-plugin-ovn ovn-source

Upgrade the charm if the option is not available:

.. code-block:: none

   juju refresh neutron-api-plugin-ovn

Procedure
---------

Set the election timer
~~~~~~~~~~~~~~~~~~~~~~

To minimise timeout issues during the upgrade, set the OVS database server
election timer to its maximum value:

.. code-block:: none

   juju config ovn-central ovsdb-server-election-timer=30

For background information, see section `Raft leader election timeout`_ of this
document.

Allow the ovn-central application to settle - use the :command:`juju status
ovn-central` command.

Ensure OVN package requirements
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Ensure that select packages are up to date on the cloud's OVN units.

Perform the package upgrades on all OVN units by running commands across the
ovn-chassis and ovn-central applications:

.. code-block:: none

   juju run -a ovn-chassis 'apt update && apt -y install \
      --only-upgrade openvswitch-common ovn-common'
   juju run -a ovn-central 'apt update && apt -y install \
      --only-upgrade openvswitch-common ovn-common'

.. note::

   Some clouds may be running ovn-dedicated-chassis as opposed to ovn-chassis.

Fail-safe mode on OVN < 22.03
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To prevent an OVN data plane outage during the upgrade to 22.03 the
``ovn-controller`` daemon must be placed into fail-safe mode. This section
corresponds to upstream's documented `fail-safe method`_.

First stop the ``ovn-northd`` daemon:

.. code-block:: none

   juju run -a ovn-central 'systemctl stop ovn-northd'

Secondly, identify the Southbound database leader unit (see the
:doc:`../../admin/networking/ovn/queries` page for guidance).

Finally, manually set the ``northd`` version to an arbitrary string. The
``ovn-controller`` processes will detect this change and adapt to be able to
understand the data that the upgraded ``northd`` daemon will subsequently
insert into the database (use the Southbound leader unit found above):

.. code-block:: none

   juju run -u <sb-db-leader-unit> 'ovn-sbctl set sb-global .  options:northd_internal_version="<string>"'

An example invocation of the above if the Southbound leader unit is
``ovn-central/2``:

.. code-block:: none

   juju run -u ovn-central/2 'ovn-sbctl set sb-global . options:northd_internal_version="safe"'

The above command contains the string 'safe'. Any string will suffice provided
that it is different from the current OVN version.

Perform the upgrade
~~~~~~~~~~~~~~~~~~~

To ensure a smooth migration, guidance is provided below that includes
verification steps.

Neutron
^^^^^^^

To upgrade OVN packages used by neutron, configure the
``neutron-api-plugin-ovn`` charm to use the overlay repository that contains
the '22.03' release of OVN:

.. code-block:: none

   juju config neutron-api-plugin-ovn ovn-source="cloud:focal-ovn-22.03"

ovn-central
^^^^^^^^^^^

Prior to upgrading the ovn-central application, change its software sources to
'distro' and change the charm's channel to '22.03/stable':

.. code-block:: none

   juju refresh ovn-central --channel 22.03/stable \
      --config <(printf "ovn-central:\n source: \"distro\"")

Now upgrade the application by selecting the UCA pocket for OVN 22.03 on Focal:

.. code-block:: none

   juju config ovn-central ovn-source=cloud:focal-ovn-22.03

As before, allow the ovn-central application to settle - use the :command:`juju
status ovn-central` command.

Verify: database migration
..........................

Ensure that the upgraded Northbound and Southbound database schemas match
what's expected (the target version). An example set of commands are provided
below.

The Northbound database's target version and actual version, respectively:

.. code-block:: none

   juju run -a ovn-central 'ovsdb-tool schema-version /usr/share/ovn/ovn-nb.ovsschema'

   Stdout: |
   6.1.0
   UnitId: ovn-central/0
   Stdout: |
   6.1.0
   UnitId: ovn-central/1
   Stdout: |
   6.1.0
   UnitId: ovn-central/2

   juju run -a ovn-central 'ovsdb-client get-schema-version unix:/var/run/ovn/ovnnb_db.sock OVN_Northbound'

   Stdout: |
   6.1.0
   UnitId: ovn-central/0
   Stdout: |
   6.1.0
   UnitId: ovn-central/1
   Stdout: |
   6.1.0
   UnitId: ovn-central/2

The Southbound database's target version and actual version, respectively:

.. code-block:: none

   juju run -a ovn-central 'ovsdb-tool schema-version /usr/share/ovn/ovn-sb.ovsschema'

   Stdout: |
   20.21.0
   UnitId: ovn-central/0
   Stdout: |
   20.21.0
   UnitId: ovn-central/2
   Stdout: |
   20.21.0
   UnitId: ovn-central/1

   juju run -a ovn-central 'ovsdb-client get-schema-version unix:/var/run/ovn/ovnsb_db.sock OVN_Southbound'

   Stdout: |
   20.21.0
   UnitId: ovn-central/0
   Stdout: |
   20.21.0
   UnitId: ovn-central/1
   Stdout: |
   20.21.0
   UnitId: ovn-central/2

If versions do not match it might be that the database migration did not
succeed (see log files under :file:`/var/log/ovn` on the ovn-central units).

Verify: cluster status
......................

Check the status of the Northbound and Southbound database clusters. It is
expected that one unit has ``Role: leader`` and the others have ``Role:
follower``. An example set of commands are provided below.

The Northbound database cluster:

.. code-block:: none

   juju run -a ovn-central 'ovs-appctl -t /var/run/ovn/ovnnb_db.ctl cluster/status OVN_Northbound' | egrep "Server ID|Role|Leader"

   Server ID: 2a92 (2a9226b6-7a57-411a-94ee-092aa6a19e40)
   Role: follower
   Leader: bc3a
   Server ID: adb2 (adb28a73-4e21-492c-81d0-f51adc6665a4)
   Role: follower
   Leader: bc3a
   Server ID: bc3a (bc3a26b1-14c0-4133-b2c3-d8f64e4b722d)
   Role: leader
   Leader: self

The Southbound database cluster:

.. code-block:: none

   juju run -a ovn-central 'ovs-appctl -t /var/run/ovn/ovnsb_db.ctl cluster/status OVN_Southbound' | egrep "Server ID|Role|Leader"

   Server ID: 8849 (8849b07b-cc32-47cf-8800-ed89fbc7db94)
   Role: follower
   Leader: fa7e
   Server ID: 50b7 (50b7f34e-b295-4329-8d29-47039f697365)
   Role: follower
   Leader: fa7e
   Server ID: fa7e (fa7e81bb-90e9-4c87-8ce4-cedcd54c6150)
   Role: leader
   Leader: self

ovn-chassis
^^^^^^^^^^^

To upgrade the ovn-chassis application, change the charm's channel to
'22.03/stable' and then select the UCA pocket for OVN 22.03 on Focal:

.. code-block:: console

   juju refresh ovn-chassis --channel 22.03/stable
   juju config ovn-chassis ovn-source=cloud:focal-ovn-22.03

Verify: network agents
......................

Ensure that all network agents are "alive" and "up":

.. code-block:: none

   openstack network agent list

Sample output:

.. code-block:: console

   +---------------+----------------------+---------------+---------------+-------+-------+-------------------------------+
   | ID            | Agent Type           | Host          | Avail... Zone | Alive | State | Binary                        |
   +---------------+----------------------+---------------+---------------+-------+-------+-------------------------------+
   | xxxx-xxxx-... | OVN Controller agent | xxxx-xxxx-... |               | :-)   | UP    | ovn-controller                |
   | xxxx-xxxx-... | OVN Metadata agent   | xxxx-xxxx-... |               | :-)   | UP    | networking-ovn-metadata-agent |
   | xxxx-xxxx-... | OVN Controller agent | xxxx-xxxx-... |               | :-)   | UP    | ovn-controller                |
   | xxxx-xxxx-... | OVN Metadata agent   | xxxx-xxxx-... |               | :-)   | UP    | networking-ovn-metadata-agent |
   +---------------+----------------------+---------------+---------------+-------+-------+-------------------------------+

Other resources
---------------

Raft leader election timeout
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The Raft leader election timeout is a crucial factor in the upgrade. It is
governed by the ovn-central charm's `ovsdb-server-election-timer`_
configuration option, whose default value is '4' (seconds).

The amount of wall clock time a database (Northbound or Southbound) cluster
leader consumes during the upgrade process cannot exceed the election timer. If
this occurs, the database unit attempting the upgrade (schema conversion) will
be evicted from the cluster, thereby preventing its results from being stored.
This scenario will lead to an endless retry loop.

Conversion happens on startup of the DB services after package upgrades. To
prevent the aforementioned retry loop, the startup scripts have a `30 second
hardcoded timeout`_. Therefore:

#. the maximum effective value for the ``ovsdb-server-election-timer`` option
   is '30'
#. an alternative upgrade path would be needed if the conversion cannot
   succeed within that maximum

There is no template answer for what the value of the option should be.
External factors (e.g. server performance characteristics, load, database
size, number of records) all have a role to play.

See the upstream `mailing list thread`_ for a discussion on the topic. Issue
`LP #2013344`_ raises concerns about the option's default value being too
small.

.. LINKS
.. _fail-safe method: https://docs.ovn.org/en/latest/intro/install/ovn-upgrades.html#fail-safe-upgrade
.. _ovsdb-server-election-timer: https://charmhub.io/ovn-central/configure?channel=22.03/stable#ovsdb-server-election-timer
.. _mailing list thread: https://mail.openvswitch.org/pipermail/ovs-discuss/2023-March/052316.html
.. _upstream OVN issue: https://bugs.launchpad.net/charm-ovn-chassis/+bug/1940043
.. _30 second hardcoded timeout: https://github.com/openvswitch/ovs/blob/07c27226ee96a3715126c50e1dbf6d8a1886b305/utilities/ovs-lib.in#L492)
.. _Juju upgrade documentation: https://juju.is/docs/juju/upgrading

.. BUGS
.. _LP #2013344: https://bugs.launchpad.net/charm-ovn-central/+bug/2013344
