:orphan:

========================
Series upgrade OpenStack
========================

This document will provide specific steps for how to perform a series upgrade
across the entirety of a Charmed OpenStack cloud.

.. warning::

   This document is based upon the foundational knowledge and guidelines set
   forth on the more general :doc:`series` page. That reference must be studied
   in-depth prior to attempting the steps outlined here. In particular, ensure
   that the :ref:`Pre-upgrade requirements <pre-upgrade_requirements>` are
   satisfied and that the :ref:`Workload specific preparations
   <workload_specific_preparations>` have been addressed during planning.

Downtime
--------

Although the goal is to minimise downtime the series upgrade process across a
cloud will nonetheless result in some level of downtime for the control plane.

When the machines associated with stateful applications such as percona-cluster
and rabbitmq-server undergo a series upgrade all cloud APIs will experience
downtime, in addition to the stateful applications themselves.

When machines associated with a single API application undergo a series upgrade
that individual API will also experience downtime. This is because it is
necessary to pause services in order to avoid race condition errors.

For those applications working in tandem with hacluster, as will be shown, some
hacluster units will need to be paused before the upgrade. One should assume
that the commencement of an outage coincides with this step (it will cause
cluster quorum heartbeats to fail and the service VIP will consequently go
offline).

Generalised OpenStack series upgrade
------------------------------------

This section will summarise the series upgrade steps in the context of specific
OpenStack applications. It is an enhancement of the :ref:`Generic series
upgrade <generic_series_upgrade>` section in the companion document.

Generally, this summary is well-suited to API applications (e.g. neutron-api,
keystone, nova-cloud-controller).

Applications for which this summary does **not** apply include:

#. those that do not require the pausing of units and where application
   leadership is irrelevant:

   * nova-compute
   * ceph-mon
   * ceph-osd

#. those that require a special upgrade workflow due to payload/upstream
   requirements:

   * percona-cluster
   * rabbitmq-server

.. note::

   Let the machine associated with the leader of the principal application be
   called the "principal leader machine" and its unit the "principal leader
   unit".

   Let the machines associated with the non-leaders of the principal
   application be be called the "principal non-leader machines" and their units
   the "principal non-leader units".

The steps are as follows:

#. Set the default series for the principal application.

#. If hacluster is used, pause the hacluster units not associated with the
   principal leader machine.

#. Pause the principal non-leader units.

#. Perform a series upgrade on each of the paused machines:

   #. Disable :ref:`Unattended upgrades <unattended_upgrades>`.

   #. Perform any pre-upgrade :ref:`workload maintenance tasks
      <workload_maintenance>`.

   #. Invoke the :command:`prepare` sub-command.

   #. Upgrade the operating system (APT commands).

   #. Perform any post-upgrade tasks at the machine/unit level.

   #. Re-enable Unattended upgrades.

   #. Reboot.

   #. Invoke the :command:`complete` sub-command.

#. Pause the principal leader unit.

#. Repeat step 4 for the paused principal leader machine.

#. Perform any remaining post-upgrade tasks.

#. Update the software sources for the principal application's machines.

Procedures
----------

The procedures are categorised based on application types. The example scenario
used throughout is a 'xenial' to 'bionic' series upgrade, within an OpenStack
release of Queens (i.e. the starting point is a UCA release of
'xenial-queens').

New default series for the model
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Ensure that any newly-created application units are based on the next series by
setting the model's default series appropriately:

.. code-block:: none

   juju model-config default-series=bionic

Stateful applications
~~~~~~~~~~~~~~~~~~~~~

This section covers the series upgrade procedure for containerised stateful
applications. These include:

* ceph-mon
* percona-cluster
* rabbitmq-server

A stateful application is one that maintains the state of various aspects of
the cloud. Clustered stateful applications, such as the ones given above,
require a quorum to function properly. Therefore, a stateful application should
not have all of its units restarted simultaneously; it must have the series of
its corresponding machines upgraded sequentially.

ceph-mon
^^^^^^^^

.. important::

   During this upgrade there will NOT be a Ceph service outage.

   The MON cluster will be maintained during the upgrade by the ceph-mon charm,
   rendering application leadership irrelevant. Notably, ceph-mon units do not
   need to be paused.

This scenario is represented by the following partial :command:`juju status`
command output:

.. code-block:: console

   App       Version  Status  Scale  Charm     Store       Channel  Rev  OS      Message
   ceph-mon  12.2.13  active      3  ceph-mon  charmstore  stable   483  ubuntu  Unit is ready and clustered

   Unit         Workload  Agent  Machine  Public address  Ports  Message
   ceph-mon/0   active    idle   0/lxd/0  10.246.114.57          Unit is ready and clustered
   ceph-mon/1   active    idle   1/lxd/0  10.246.114.56          Unit is ready and clustered
   ceph-mon/2*  active    idle   2/lxd/0  10.246.114.26          Unit is ready and clustered

#. Perform any workload maintenance pre-upgrade steps.

   For ceph-mon, there are no recommended steps to take.

#. Set the default series for the principal application:

   .. code-block:: none

      juju set-series ceph-mon bionic

#. Perform a series upgrade of the machines in any order:

   .. code-block:: none

      juju upgrade-series 0/lxd/0 prepare bionic
      juju ssh 0/lxd/0 sudo apt update
      juju ssh 0/lxd/0 sudo apt full-upgrade
      juju ssh 0/lxd/0 sudo do-release-upgrade

   For ceph-mon, there are no post-upgrade steps; the prompt to reboot can be
   answered in the affirmative.

   Invoke the :command:`complete` sub-command:

   .. code-block:: none

      juju upgrade-series 0/lxd/0 complete

#. Repeat step 4 for each of the remaining machines:

   .. code-block:: none

      juju upgrade-series 1/lxd/0 prepare bionic
      juju ssh 1/lxd/0 sudo apt update
      juju ssh 1/lxd/0 sudo apt full-upgrade
      juju ssh 1/lxd/0 sudo do-release-upgrade  # and reboot
      juju upgrade-series 1/lxd/0 complete

   .. code-block:: none

      juju upgrade-series 2/lxd/0 prepare bionic
      juju ssh 2/lxd/0 sudo apt update
      juju ssh 2/lxd/0 sudo apt full-upgrade
      juju ssh 2/lxd/0 sudo do-release-upgrade  # and reboot
      juju upgrade-series 2/lxd/0 complete

#. Perform any remaining post-upgrade tasks.

   For ceph-mon, there are no remaining post-upgrade steps.

#. Update the software sources for the application's machines.

   For ceph-mon, set the value of the ``source`` configuration option to
   'distro':

   .. code-block:: none

      juju config ceph-mon source=distro

The final partial :command:`juju status` output looks like this:

.. code-block:: console

   App       Version  Status  Scale  Charm     Store       Channel  Rev  OS      Message
   ceph-mon  12.2.13  active      3  ceph-mon  charmstore  stable   483  ubuntu  Unit is ready and clustered

   Unit         Workload  Agent  Machine  Public address  Ports  Message
   ceph-mon/0   active    idle   0/lxd/0  10.246.114.57          Unit is ready and clustered
   ceph-mon/1   active    idle   1/lxd/0  10.246.114.56          Unit is ready and clustered
   ceph-mon/2*  active    idle   2/lxd/0  10.246.114.26          Unit is ready and clustered

Note that the version of Ceph has not been upgraded (from 12.2.13 - Luminous)
since the OpenStack release (of Queens) remains unchanged.

rabbitmq-server
^^^^^^^^^^^^^^^

To ensure proper cluster health, the RabbitMQ cluster is not reformed until all
rabbitmq-server units are series upgraded. An action is then used to complete
the upgrade by bringing the cluster back online.

.. warning::

   During this upgrade there will be a RabbitMQ service outage.

This scenario is represented by the following partial :command:`juju status`
command output:

.. code-block:: console

   App              Version  Status  Scale  Charm            Store       Channel  Rev  OS      Message
   rabbitmq-server  3.5.7    active      3  rabbitmq-server  charmstore  stable   118  ubuntu  Unit is ready and clustered

   Unit                Workload  Agent  Machine  Public address  Ports     Message
   rabbitmq-server/0*  active    idle   0/lxd/0  10.0.0.162      5672/tcp  Unit is ready and clustered
   rabbitmq-server/1   active    idle   1/lxd/0  10.0.0.164      5672/tcp  Unit is ready and clustered
   rabbitmq-server/2   active    idle   2/lxd/0  10.0.0.163      5672/tcp  Unit is ready and clustered

In summary, the principal leader unit is rabbitmq-server/0 and is deployed on
machine 0/lxd/0 (the principal leader machine).

#. Perform any workload maintenance pre-upgrade steps.

   For rabbitmq-server, there are no recommended steps to take.

#. Set the default series for the principal application:

   .. code-block:: none

      juju set-series rabbitmq-server bionic

#. Pause the principal non-leader units:

   .. code-block:: none

      juju run-action --wait rabbitmq-server/1 pause
      juju run-action --wait rabbitmq-server/2 pause

#. Perform a series upgrade of the principal leader machine:

   .. code-block:: none

      juju upgrade-series 0/lxd/0 prepare bionic
      juju ssh 0/lxd/0 sudo apt update
      juju ssh 0/lxd/0 sudo apt full-upgrade
      juju ssh 0/lxd/0 sudo do-release-upgrade

   For rabbitmq-server, there are no post-upgrade steps; the prompt to reboot
   can be answered in the affirmative.

   Invoke the :command:`complete` sub-command:

   .. code-block:: none

      juju upgrade-series 0/lxd/0 complete

#. Repeat step 4 for each of the principal non-leader machines:

   .. code-block:: none

      juju upgrade-series 1/lxd/0 prepare bionic
      juju ssh 1/lxd/0 sudo apt update
      juju ssh 1/lxd/0 sudo apt full-upgrade
      juju ssh 1/lxd/0 sudo do-release-upgrade  # and reboot
      juju upgrade-series 1/lxd/0 complete

   .. code-block:: none

      juju upgrade-series 2/lxd/0 prepare bionic
      juju ssh 2/lxd/0 sudo apt update
      juju ssh 2/lxd/0 sudo apt full-upgrade
      juju ssh 2/lxd/0 sudo do-release-upgrade  # and reboot
      juju upgrade-series 2/lxd/0 complete

#. Perform any remaining post-upgrade tasks.

   For rabbitmq-server, run an action:

   .. code-block:: none

      juju run-action --wait rabbitmq-server/leader complete-cluster-series-upgrade

#. Update the software sources for the application's machines.

   For rabbitmq-server, set the value of the ``source`` configuration option to
   'distro':

   .. code-block:: none

      juju config rabbitmq-server source=distro

The final partial :command:`juju status` output looks like this:

.. code-block:: console

   App              Version  Status  Scale  Charm            Store       Channel  Rev  OS      Message
   rabbitmq-server  3.6.10   active      3  rabbitmq-server  charmstore  stable   118  ubuntu  Unit is ready and clustered

   Unit                Workload  Agent  Machine  Public address  Ports     Message
   rabbitmq-server/0*  active    idle   0/lxd/0  10.0.0.162      5672/tcp  Unit is ready and clustered
   rabbitmq-server/1   active    idle   1/lxd/0  10.0.0.164      5672/tcp  Unit is ready and clustered
   rabbitmq-server/2   active    idle   2/lxd/0  10.0.0.163      5672/tcp  Unit is ready and clustered

Note that the version of RabbitMQ has been upgraded (from 3.5.7 to 3.6.10)
since more recent software has been found in the Ubuntu package archive for
Bionic.

percona-cluster
^^^^^^^^^^^^^^^

.. warning::

   During this upgrade there will be a MySQL service outage.

.. note::

   These upstream resources may also be useful:

   * `Upgrading Percona XtraDB Cluster`_
   * `Percona XtraDB Cluster In-Place Upgrading Guide From 5.5 to 5.6`_
   * `Galera replication - how to recover a PXC cluster`_

To ensure proper cluster health, the Percona cluster is not reformed until all
percona-cluster units are series upgraded. An action is then used to complete
the upgrade by bringing the cluster back online.

.. warning::

   The eoan series is the last series supported by the percona-cluster charm.
   It is replaced by the `mysql-innodb-cluster`_ and `mysql-router`_ charms in
   the focal series. The migration steps are documented in
   :doc:`../../project/procedures/percona-series-upgrade-to-focal`.

   Do not upgrade the machines hosting percona-cluster units to the focal
   series. To be clear, if percona-cluster is containerised then it is the LXD
   container that must not be upgraded.

This scenario is represented by the following partial :command:`juju status`
command output:

.. code-block:: console

   App                        Version  Status  Scale  Charm            Store       Channel  Rev  OS      Message
   percona-cluster            5.6.37   active      3  percona-cluster  charmstore  stable   302  ubuntu  Unit is ready
   percona-cluster-hacluster           active      3  hacluster        charmstore  stable    81  ubuntu  Unit is ready and clustered

   Unit                            Workload  Agent  Machine  Public address  Ports     Message
   percona-cluster/0*              active    idle   0/lxd/1  10.0.0.165      3306/tcp  Unit is ready
     percona-cluster-hacluster/2   active    idle            10.0.0.165                Unit is ready and clustered
   percona-cluster/1               active    idle   1/lxd/1  10.0.0.166      3306/tcp  Unit is ready
     percona-cluster-hacluster/0*  active    idle            10.0.0.166                Unit is ready and clustered
   percona-cluster/2               active    idle   2/lxd/1  10.0.0.167      3306/tcp  Unit is ready
     percona-cluster-hacluster/1   active    idle            10.0.0.167                Unit is ready and clustered

In summary, the principal leader unit is percona-cluster/0 and is deployed on
machine 0/lxd/1 (the principal leader machine).

#. Perform any workload maintenance pre-upgrade steps.

   For percona-cluster, take a backup and transfer it to a secure location:

   .. code-block:: none

      juju run-action --wait percona-cluster/leader backup
      juju scp -- -r percona-cluster/leader:/opt/backups/mysql /path/to/local/directory

   Permissions will need to be altered on the remote machine, and note that the
   :command:`scp` command transfers **all** existing backups.

#. Set the default series for the principal application:

   .. code-block:: none

      juju set-series percona-cluster bionic

#. Pause the hacluster units not associated with the principal leader machine:

   .. code-block:: none

      juju run-action --wait percona-cluster-hacluster/0 pause
      juju run-action --wait percona-cluster-hacluster/1 pause

#. Pause the principal non-leader units:

   .. code-block:: none

      juju run-action --wait percona-cluster/1 pause
      juju run-action --wait percona-cluster/2 pause

   Leaving the principal leader unit up will ensure it has the latest MySQL
   sequence number; it will be considered the most up to date cluster member.

   At this point the partial :command:`juju status` output looks like this:

   .. code-block:: console

      App                        Version  Status       Scale  Charm            Store       Channel  Rev  OS      Message
      percona-cluster            5.6.37   maintenance      3  percona-cluster  charmstore  stable   302  ubuntu  Paused. Use 'resume' action to resume normal service.
      percona-cluster-hacluster           maintenance      3  hacluster        charmstore  stable    81  ubuntu  Paused. Use 'resume' action to resume normal service.

      Unit                            Workload     Agent  Machine  Public address  Ports     Message
      percona-cluster/0*              active       idle   0/lxd/1  10.0.0.165      3306/tcp  Unit is ready
        percona-cluster-hacluster/2   active       idle            10.0.0.165                Unit is ready and clustered
      percona-cluster/1               maintenance  idle   1/lxd/1  10.0.0.166      3306/tcp  Paused. Use 'resume' action to resume normal service.
        percona-cluster-hacluster/0*  maintenance  idle            10.0.0.166                Paused. Use 'resume' action to resume normal service.
      percona-cluster/2               maintenance  idle   2/lxd/1  10.0.0.167      3306/tcp  Paused. Use 'resume' action to resume normal service.
        percona-cluster-hacluster/1   maintenance  idle            10.0.0.167                Paused. Use 'resume' action to resume normal service.

#. Perform a series upgrade of the principal leader machine:

   .. code-block:: none

      juju upgrade-series 0/lxd/1 prepare bionic
      juju ssh 0/lxd/1 sudo apt update
      juju ssh 0/lxd/1 sudo apt full-upgrade
      juju ssh 0/lxd/1 sudo do-release-upgrade

   For percona-cluster, there are no post-upgrade steps; the prompt to reboot
   can be answered in the affirmative.

   Invoke the :command:`complete` sub-command:

   .. code-block:: none

      juju upgrade-series 0/lxd/1 complete

#. Repeat step 4 for each of the principal non-leader machines:

   .. code-block:: none

      juju upgrade-series 1/lxd/1 prepare bionic
      juju ssh 1/lxd/1 sudo apt update
      juju ssh 1/lxd/1 sudo apt full-upgrade
      juju ssh 1/lxd/1 sudo do-release-upgrade  # and reboot
      juju upgrade-series 1/lxd/1 complete

   .. code-block:: none

      juju upgrade-series 2/lxd/1 prepare bionic
      juju ssh 2/lxd/1 sudo apt update
      juju ssh 2/lxd/1 sudo apt full-upgrade
      juju ssh 2/lxd/1 sudo do-release-upgrade  # and reboot
      juju upgrade-series 2/lxd/1 complete

#. Perform any remaining post-upgrade tasks.

   For percona-cluster, a sanity check should be performed on the leader unit's
   databases and data.

   Also, an action must be run:

   .. code-block:: none

      juju run-action --wait percona-cluster/leader complete-cluster-series-upgrade

#. Update the software sources for the application's machines.

   For percona-cluster, set the value of the ``source`` configuration option to
   'distro':

   .. code-block:: none

      juju config percona-cluster source=distro

The final partial :command:`juju status` output looks like this:

.. code-block:: console

   App                        Version  Status  Scale  Charm            Store       Channel  Rev  OS      Message
   percona-cluster            5.7.20   active      3  percona-cluster  charmstore  stable   302  ubuntu  Unit is ready
   percona-cluster-hacluster           active      3  hacluster        charmstore  stable    81  ubuntu  Unit is ready and clustered

   Unit                            Workload  Agent  Machine  Public address  Ports     Message
   percona-cluster/0*              active    idle   0/lxd/1  10.0.0.165      3306/tcp  Unit is ready
     percona-cluster-hacluster/2   active    idle            10.0.0.165                Unit is ready and clustered
   percona-cluster/1               active    idle   1/lxd/1  10.0.0.166      3306/tcp  Unit is ready
     percona-cluster-hacluster/0*  active    idle            10.0.0.166                Unit is ready and clustered
   percona-cluster/2               active    idle   2/lxd/1  10.0.0.167      3306/tcp  Unit is ready
     percona-cluster-hacluster/1   active    idle            10.0.0.167                Unit is ready and clustered

Note that the version of Percona has been upgraded (from 5.6.37 to 5.7.20)
since more recent software has been found in the Ubuntu package archive for
Bionic.

API applications
~~~~~~~~~~~~~~~~

This section covers series upgrade procedures for containerised API
applications. These include, but are not limited to:

* cinder
* glance
* keystone
* neutron-api
* nova-cloud-controller

Machines hosting API applications can have their series upgraded concurrently
because those applications are stateless. This results in a dramatically
reduced downtime for the application. A sequential approach will not reduce
downtime as the HA services will still need to be brought down during the
upgrade associated with the application leader.

The following two sub-sections will show how to perform a series upgrade
concurrently for a single API application and for multiple API applications.

Upgrading a single API application concurrently
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This example procedure will be based on the keystone application.

This scenario is represented by the following partial :command:`juju status`
command output:

.. code-block:: console

   App                 Version  Status  Scale  Charm      Store       Channel  Rev  OS      Message
   keystone            13.0.4   active      3  keystone   charmstore  stable   330  ubuntu  Application Ready
   keystone-hacluster           active      3  hacluster  charmstore  stable    81  ubuntu  Unit is ready and clustered

   Unit                     Workload  Agent  Machine  Public address  Ports     Message
   keystone/0*              active    idle   0/lxd/0  10.0.0.198      5000/tcp  Unit is ready
     keystone-hacluster/2   active    idle            10.0.0.198                Unit is ready and clustered
   keystone/1               active    idle   1/lxd/0  10.0.0.196      5000/tcp  Unit is ready
     keystone-hacluster/0*  active    idle            10.0.0.196                Unit is ready and clustered
   keystone/2               active    idle   2/lxd/0  10.0.0.197      5000/tcp  Unit is ready
     keystone-hacluster/1   active    idle            10.0.0.197                Unit is ready and clustered

In summary, the principal leader unit is keystone/0 and is deployed on machine
0/lxd/0 (the principal leader machine).

#. Set the default series for the principal application:

   .. code-block:: none

      juju set-series keystone bionic

#. Pause the hacluster units not associated with the principal leader machine:

   .. code-block:: none

      juju run-action --wait keystone-hacluster/0 pause
      juju run-action --wait keystone-hacluster/1 pause

#. Pause the principal non-leader units:

   .. code-block:: none

      juju run-action --wait keystone/1 pause
      juju run-action --wait keystone/2 pause

#. Perform any workload maintenance pre-upgrade steps on all machines. There
   are no keystone-specific steps to perform.

#. Invoke the :command:`prepare` sub-command on all machines, **starting with
   the principal leader machine**:

   .. code-block:: none

      juju upgrade-series 0/lxd/0 prepare bionic
      juju upgrade-series 1/lxd/0 prepare bionic
      juju upgrade-series 2/lxd/0 prepare bionic

   At this point the :command:`juju status` output looks like this:

   .. code-block:: console

      App                 Version  Status   Scale  Charm      Store       Channel  Rev  OS      Message
      keystone            13.0.4   blocked      3  keystone   charmstore  stable   330  ubuntu  Unit paused.
      keystone-hacluster           blocked      3  hacluster  charmstore  stable    81  ubuntu  Ready for do-release-upgrade. Set complete when finished

      Unit                     Workload  Agent  Machine  Public address  Ports     Message
      keystone/0*              blocked   idle   0/lxd/0  10.0.0.198      5000/tcp  Ready for do-release-upgrade and reboot. Set complete when finished., Unit paused.
        keystone-hacluster/2   blocked   idle            10.0.0.198                Ready for do-release-upgrade. Set complete when finished
      keystone/1               blocked   idle   1/lxd/0  10.0.0.196      5000/tcp  Ready for do-release-upgrade and reboot. Set complete when finished., Unit paused.
        keystone-hacluster/0*  blocked   idle            10.0.0.196                Ready for do-release-upgrade. Set complete when finished
      keystone/2               blocked   idle   2/lxd/0  10.0.0.197      5000/tcp  Ready for do-release-upgrade and reboot. Set complete when finished., Unit paused.
        keystone-hacluster/1   blocked   idle            10.0.0.197                Ready for do-release-upgrade. Set complete when finished

#. Upgrade the operating system on all machines. The non-interactive method is
   used here:

   .. code-block:: none

      juju run --machine=0/lxd/0,1/lxd/0,2/lxd/0 --timeout=10m \
         -- sudo apt-get update

      juju run --machine=0/lxd/0,1/lxd/0,2/lxd/0 --timeout=60m \
         -- sudo DEBIAN_FRONTEND=noninteractive apt-get --assume-yes \
         -o "Dpkg::Options::=--force-confdef" \
         -o "Dpkg::Options::=--force-confold" dist-upgrade

      juju run --machine=0/lxd/0,1/lxd/0,2/lxd/0 --timeout=120m \
         -- sudo DEBIAN_FRONTEND=noninteractive \
         do-release-upgrade -f DistUpgradeViewNonInteractive

   .. important::

      Choose values for the ``--timeout`` option that are appropriate for the
      task at hand.

#. Perform any post-upgrade tasks.

   For keystone, there are no specific steps to perform.

#. Reboot all machines:

   .. code-block:: none

      juju run --machine=0/lxd/0,1/lxd/0,2/lxd/0 -- sudo reboot

#. Invoke the :command:`complete` sub-command on all machines:

   .. code-block:: none

      juju upgrade-series 0/lxd/0 complete
      juju upgrade-series 1/lxd/0 complete
      juju upgrade-series 2/lxd/0 complete

#. Perform any remaining post-upgrade tasks.

   For keystone, there are no remaining post-upgrade steps.

#. Update the software sources for the application's machines.

   For keystone, set the value of the ``openstack-origin`` configuration option
   to 'distro':

   .. code-block:: none

      juju config keystone openstack-origin=distro

The final partial :command:`juju status` output looks like this:

.. code-block:: console

   App                 Version  Status  Scale  Charm      Store       Channel  Rev  OS      Message
   keystone            13.0.4   active      3  keystone   charmstore  stable   330  ubuntu  Application Ready
   keystone-hacluster           active      3  hacluster  charmstore  stable    81  ubuntu  Unit is ready and clustered

   Unit                     Workload  Agent  Machine  Public address  Ports     Message
   keystone/0*              active    idle   0/lxd/0  10.0.0.198      5000/tcp  Unit is ready
     keystone-hacluster/2   active    idle            10.0.0.198                Unit is ready and clustered
   keystone/1               active    idle   1/lxd/0  10.0.0.196      5000/tcp  Unit is ready
     keystone-hacluster/0*  active    idle            10.0.0.196                Unit is ready and clustered
   keystone/2               active    idle   2/lxd/0  10.0.0.197      5000/tcp  Unit is ready
     keystone-hacluster/1   active    idle            10.0.0.197                Unit is ready and clustered

Note that the version of Keystone has not been upgraded (from 13.0.4) since the
OpenStack release (of Queens) remains unchanged.

Upgrading multiple API applications concurrently
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This example procedure will be based on the nova-cloud-controller and glance
applications.

This scenario is represented by the following partial :command:`juju status`
command output:

.. code-block:: console

   App                              Version  Status  Scale  Charm                  Store       Channel  Rev  OS      Message
   glance                           16.0.1   active      3  glance                 charmstore  stable   484  ubuntu  Unit is ready
   glance-hacluster                          active      3  hacluster              charmstore  stable    81  ubuntu  Unit is ready and clustered
   nova-cloud-controller            17.0.13  active      3  nova-cloud-controller  charmstore  stable   555  ubuntu  Unit is ready
   nova-cloud-controller-hacluster           active      3  hacluster              charmstore  stable    81  ubuntu  Unit is ready and clustered

   Unit                                  Workload  Agent  Machine  Public address  Ports              Message
   glance/0*                             active    idle   2/lxd/1  10.246.114.27   9292/tcp           Unit is ready
     glance-hacluster/0*                 active    idle            10.246.114.27                      Unit is ready and clustered
   glance/1                              active    idle   2/lxd/3  10.246.114.64   9292/tcp           Unit is ready
     glance-hacluster/2                  active    idle            10.246.114.64                      Unit is ready and clustered
   glance/2                              active    idle   1/lxd/4  10.246.114.65   9292/tcp           Unit is ready
     glance-hacluster/1                  active    idle            10.246.114.65                      Unit is ready and clustered
   nova-cloud-controller/0*              active    idle   2/lxd/2  10.246.114.25   8774/tcp,8778/tcp  Unit is ready
     nova-cloud-controller-hacluster/0*  active    idle            10.246.114.25                      Unit is ready and clustered
   nova-cloud-controller/1               active    idle   1/lxd/2  10.246.114.61   8774/tcp,8778/tcp  Unit is ready
     nova-cloud-controller-hacluster/1   active    idle            10.246.114.61                      Unit is ready and clustered
   nova-cloud-controller/2               active    idle   0/lxd/4  10.246.114.62   8774/tcp,8778/tcp  Unit is ready
     nova-cloud-controller-hacluster/2   active    idle            10.246.114.62                      Unit is ready and clustered

In summary,

* The glance principal leader unit is glance/0 and is deployed on machine
  2/lxd/1 (the glance principal leader machine).
* The nova-cloud-controller principal leader unit is nova-cloud-controller/0
  and is deployed on machine 2/lxd/2 (the nova-cloud-controller principal
  leader machine).

#. Set the default series for the principal applications:

   .. code-block:: none

      juju set-series glance bionic
      juju set-series nova-cloud-controller bionic

#. Pause the hacluster units not associated with their principal leader
   machines:

   .. code-block:: none

      juju run-action --wait glance-hacluster/1 pause
      juju run-action --wait glance-hacluster/2 pause
      juju run-action --wait nova-cloud-controller-hacluster/1 pause
      juju run-action --wait nova-cloud-controller-hacluster/2 pause

#. Pause the principal non-leader units:

   .. code-block:: none

      juju run-action --wait glance/1 pause
      juju run-action --wait glance/2 pause
      juju run-action --wait nova-cloud-controller/1 pause
      juju run-action --wait nova-cloud-controller/2 pause

#. Perform any workload maintenance pre-upgrade steps on all machines. There
   are no glance-specific nor nova-cloud-controller-specific steps to perform.

#. Invoke the :command:`prepare` sub-command on all machines, **starting with
   the principal leader machines**. The procedure has been expedited slightly
   by adding the ``--yes`` confirmation option:

   .. code-block:: none

      juju upgrade-series --yes 2/lxd/1 prepare bionic
      juju upgrade-series --yes 2/lxd/2 prepare bionic
      juju upgrade-series --yes 2/lxd/3 prepare bionic
      juju upgrade-series --yes 1/lxd/4 prepare bionic
      juju upgrade-series --yes 1/lxd/2 prepare bionic
      juju upgrade-series --yes 0/lxd/4 prepare bionic

#. Upgrade the operating system on all machines. The non-interactive method is
   used here:

   .. code-block:: none

      juju run --machine=2/lxd/1,2/lxd/2,2/lxd/3,1/lxd/4,1/lxd/2,0/lxd/4 \
         --timeout=20m -- sudo apt-get update

      juju run --machine=2/lxd/1,2/lxd/2,2/lxd/3,1/lxd/4,1/lxd/2,0/lxd/4 \
         --timeout=120m -- sudo DEBIAN_FRONTEND=noninteractive apt-get --assume-yes \
         -o "Dpkg::Options::=--force-confdef" \
         -o "Dpkg::Options::=--force-confold" dist-upgrade

      juju run --machine=2/lxd/1,2/lxd/2,2/lxd/3,1/lxd/4,1/lxd/2,0/lxd/4 \
         --timeout=240m -- sudo DEBIAN_FRONTEND=noninteractive \
         do-release-upgrade -f DistUpgradeViewNonInteractive

#. Perform any workload maintenance post-upgrade steps on all machines. There
   are no glance-specific or nova-cloud-controller-specific steps to perform.

#. Reboot all machines:

   .. code-block:: none

      juju run --machine=2/lxd/1,2/lxd/2,2/lxd/3,1/lxd/4,1/lxd/2,0/lxd/4 \
         -- sudo reboot

#. Invoke the :command:`complete` sub-command on all machines:

   .. code-block:: none

      juju upgrade-series 2/lxd/1 complete
      juju upgrade-series 2/lxd/2 complete
      juju upgrade-series 2/lxd/3 complete
      juju upgrade-series 1/lxd/4 complete
      juju upgrade-series 1/lxd/2 complete
      juju upgrade-series 0/lxd/4 complete

#. Update the software sources for the application's machines.

   For glance and nova-cloud-controller, set the value of the
   ``openstack-origin`` configuration option to 'distro':

   .. code-block:: none

      juju config glance openstack-origin=distro
      juju config nova-cloud-controller openstack-origin=distro

The final partial :command:`juju status` output looks like this:

.. code-block:: console

   App                              Version  Status  Scale  Charm                  Store       Channel  Rev  OS      Message
   glance                           16.0.1   active      3  glance                 charmstore  stable   484  ubuntu  Unit is ready
   glance-hacluster                          active      3  hacluster              charmstore  stable    81  ubuntu  Unit is ready and clustered
   nova-cloud-controller            17.0.13  active      3  nova-cloud-controller  charmstore  stable   555  ubuntu  Unit is ready
   nova-cloud-controller-hacluster           active      3  hacluster              charmstore  stable    81  ubuntu  Unit is ready and clustered

   Unit                                  Workload  Agent  Machine  Public address  Ports              Message
   glance/0*                             active    idle   2/lxd/1  10.246.114.27   9292/tcp           Unit is ready
     glance-hacluster/0*                 active    idle            10.246.114.27                      Unit is ready and clustered
   glance/1                              active    idle   2/lxd/3  10.246.114.64   9292/tcp           Unit is ready
     glance-hacluster/2                  active    idle            10.246.114.64                      Unit is ready and clustered
   glance/2                              active    idle   1/lxd/4  10.246.114.65   9292/tcp           Unit is ready
     glance-hacluster/1                  active    idle            10.246.114.65                      Unit is ready and clustered
   nova-cloud-controller/0*              active    idle   2/lxd/2  10.246.114.25   8774/tcp,8778/tcp  Unit is ready
     nova-cloud-controller-hacluster/0*  active    idle            10.246.114.25                      Unit is ready and clustered
   nova-cloud-controller/1               active    idle   1/lxd/2  10.246.114.61   8774/tcp,8778/tcp  Unit is ready
     nova-cloud-controller-hacluster/1   active    idle            10.246.114.61                      Unit is ready and clustered
   nova-cloud-controller/2               active    idle   0/lxd/4  10.246.114.62   8774/tcp,8778/tcp  Unit is ready
     nova-cloud-controller-hacluster/2   active    idle            10.246.114.62                      Unit is ready and clustered

Physical machines
~~~~~~~~~~~~~~~~~

This section looks at series upgrades from the standpoint of an individual
(physical) machine. This is different from looking at series upgrades from the
standpoint of applications that happen to be running on certain machines.

Since the standard topology for Charmed OpenStack is to optimise
containerisation (with one service per container), a physical machine is
expected to directly host only those applications which cannot generally be
containerised. These notably include:

* ceph-osd
* neutron-gateway
* nova-compute

Naturally, when the physical machine is rebooted all containerised applications
will also go offline.

It is assumed that all affected services, as much as is possible, are under
HA. Note that a hypervisor (nova-compute) cannot be made highly available.

When performing a series upgrade on a physical machine more attention should be
accorded to workload maintenance pre-upgrade steps:

* For compute nodes migrate all running VMs to another hypervisor.
* For network nodes migrate routers to another cloud node.
* Any storage related tasks that may be required.
* Any site specific tasks that may be required.

The following two sub-sections will examine series upgrades for a single
physical machine and, concurrently, for multiple physical machines.

Upgrading a single physical machine
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This scenario is represented by the following partial :command:`juju status`
command output:

.. code-block:: console

   App                              Version  Status  Scale  Charm                  Store       Channel  Rev  OS      Message
   ceph-mon                         12.2.13  active      1  ceph-mon               charmstore  stable   483  ubuntu  Unit is ready and clustered
   ceph-osd                         12.2.13  active      1  ceph-osd               charmstore  stable   502  ubuntu  Unit is ready (1 OSD)
   glance                           16.0.1   active      1  glance                 charmstore  stable   484  ubuntu  Unit is ready
   glance-hacluster                          active      0  hacluster              charmstore  stable    81  ubuntu  Unit is ready and clustered
   nova-cloud-controller            17.0.13  active      1  nova-cloud-controller  charmstore  stable   555  ubuntu  Unit is ready
   nova-cloud-controller-hacluster           active      0  hacluster              charmstore  stable    81  ubuntu  Unit is ready and clustered
   nova-compute                     17.0.13  active      1  nova-compute           charmstore  stable   578  ubuntu  Unit is ready

   Unit                                 Workload  Agent  Machine  Public address  Ports              Message
   ceph-mon/1                           active    idle   1/lxd/0  10.246.114.56                      Unit is ready and clustered
   ceph-osd/1                           active    idle   1        10.246.114.22                      Unit is ready (1 OSD)
   glance/2                             active    idle   1/lxd/4  10.246.114.65   9292/tcp           Unit is ready
     glance-hacluster/1                 active    idle            10.246.114.65                      Unit is ready and clustered
   nova-cloud-controller/1              active    idle   1/lxd/2  10.246.114.61   8774/tcp,8778/tcp  Unit is ready
     nova-cloud-controller-hacluster/1  active    idle            10.246.114.61                      Unit is ready and clustered
   nova-compute/0*                      active    idle   1        10.246.114.22                      Unit is ready
     neutron-openvswitch/0*             active    idle            10.246.114.22                      Unit is ready

   Machine  State    DNS            Inst id              Series  AZ       Message
   1        started  10.246.114.22  node-fontana         xenial  default  Deployed
   1/lxd/0  started  10.246.114.56  juju-0642e9-1-lxd-0  bionic  default  series upgrade completed: success
   1/lxd/2  started  10.246.114.61  juju-0642e9-1-lxd-2  bionic  default  series upgrade completed: success
   1/lxd/4  started  10.246.114.65  juju-0642e9-1-lxd-4  bionic  default  series upgrade completed: success

As is evidenced by the noted series for each Juju machine, only the physical
machine remains to have its series upgraded. This example procedure will
therefore involve the nova-compute and ceph-osd applications. Note however that
the nova-compute application is coupled with the neutron-openvswitch
subordinate application.

Discarding those applications whose machines have already been upgraded we
arrive at the following output:

.. code-block:: console

   App                              Version  Status  Scale  Charm                  Store       Channel  Rev  OS      Message
   ceph-osd                         12.2.13  active      1  ceph-osd               charmstore  stable   502  ubuntu  Unit is ready (1 OSD)
   neutron-openvswitch              12.1.1   active      0  neutron-openvswitch    charmstore  stable   454  ubuntu  Unit is ready
   nova-compute                     17.0.13  active      1  nova-compute           charmstore  stable   578  ubuntu  Unit is ready

   Unit                                 Workload  Agent  Machine  Public address  Ports              Message
   ceph-osd/1                           active    idle   1        10.246.114.22                      Unit is ready (1 OSD)
   nova-compute/0*                      active    idle   1        10.246.114.22                      Unit is ready
     neutron-openvswitch/0*             active    idle            10.246.114.22                      Unit is ready

In summary, the ceph-osd and nova-compute applications are hosted on machine 1.
Since application leadership does not play a significant role with these two
applications, and because the hacluster application is not present, there will
be no units to pause.

.. important::

   As was the case for the upgrade procedure involving the ceph-mon
   application, during the upgrade involving ceph-osd, there will NOT be a Ceph
   service outage.

#. It is recommended to set the Ceph cluster OSDs to 'noout' to prevent the
   rebalancing of data. This is typically done at the application level (i.e.
   not at the unit or machine level):

   .. code-block:: none

      juju run-action --wait ceph-mon/leader set-noout

#. Perform any workload maintenance pre-upgrade steps.

   All running VMs should be migrated to another hypervisor. See cloud
   operation :doc:`../ops-live-migrate-vms`.

#. Perform a series upgrade of the machine:

   .. code-block:: none

      juju upgrade-series 1 prepare bionic
      juju ssh 1 sudo apt update
      juju ssh 1 sudo apt full-upgrade
      juju ssh 1 sudo do-release-upgrade  # and reboot
      juju upgrade-series 1 complete

#. Perform any remaining post-upgrade tasks.

   If OSDs were previously set to 'noout' then verify the up/in status of the
   OSDs and then unset 'noout' for the cluster:

   .. code-block:: none

      juju run --unit ceph-mon/leader -- ceph status
      juju run-action --wait ceph-mon/leader unset-noout

#. Update the software sources for the machine.

   .. caution::

      As was done in previous procedures, only set software sources once all
      machines for the associated applications have had their series upgraded.

   For the principal applications ceph-osd and nova-compute, set the
   appropriate configuration option to 'distro':

   .. code-block:: none

      juju config nova-compute openstack-origin=distro
      juju config ceph-osd source=distro

   .. note::

      Although updating the software sources more than once on the same machine
      may appear redundant it is recommended to do so.

Upgrading multiple physical hosts concurrently
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When physical machines have their series upgraded concurrently Availability
Zones need to be taken into account. Machines should be placed into upgrade
groups such that any API services running on them have a maximum of one unit
per group. This is to ensure API availability at the reboot stage.

This simplified bundle is used to demonstrate the general idea:

.. code-block:: yaml

   series: xenial
   machines:
     0: {}
     1: {}
     2: {}
     3: {}
     4: {}
     5: {}
   applications:
     nova-compute:
       charm: cs:nova-compute
       num_units: 3
       options:
         openstack-origin: cloud:xenial-queens
       to:
         - 0
         - 2
         - 4
     keystone:
       charm: cs:keystone
       num_units: 3
       options:
         vip: 10.85.132.200
         openstack-origin: cloud:xenial-queens
       to:
         - lxd:1
         - lxd:3
         - lxd:5
     keystone-hacluster:
       charm: cs:hacluster
       options:
         cluster_count: 3

Three upgrade groups could consist of the following machines:

#. Machines 0 and 1
#. Machines 2 and 3
#. Machines 4 and 5

In this way, a less time-consuming series upgrade can be performed while still
ensuring the availability of services.

.. caution::

   For the ceph-osd application, ensure that rack-aware replication rules exist
   in the CRUSH map if machines are being rebooted together. This is to prevent
   significant interruption to running workloads from occurring if the
   same placement group is hosted on those machines. For example, if ceph-mon
   is deployed with ``customize-failure-domain`` set to 'true' and the ceph-osd
   units are hosted on machines in three or more separate Juju AZs you can
   safely reboot ceph-osd machines simultaneously in the same zone. See the
   :ref:`ha_ceph_az` section on the :doc:`../ha` page for details.

Automation
----------

Series upgrades across an OpenStack cloud can be time consuming, even when
using concurrent methods wherever possible. They can also be tedious and thus
susceptible to human error.

The following code examples encapsulate the processes described in this
document. They are provided solely to illustrate the methods used to develop
and test the series upgrade primitives:

* `Parallel tests`_: An example that is used as a functional verification of
  a series upgrade in the OpenStack Charms project. Search for function
  ``test_200_run_series_upgrade``.
* `Upgrade helpers`_: A set of helpers used in the above upgrade example.

.. caution::

   The example code should only be used for its intended use case of
   development and testing. Do not attempt to automate a series upgrade on a
   production cloud.

.. LINKS
.. _Parallel tests: https://github.com/openstack-charmers/zaza-openstack-tests/blob/master/zaza/openstack/charm_tests/series_upgrade/parallel_tests.py
.. _Upgrade helpers: https://github.com/openstack-charmers/zaza-openstack-tests/blob/master/zaza/openstack/utilities/parallel_series_upgrade.py
.. _Upgrading Percona XtraDB Cluster: https://www.percona.com/doc/percona-xtradb-cluster/LATEST/howtos/upgrade_guide.html
.. _Percona XtraDB Cluster In-Place Upgrading Guide From 5.5 to 5.6: https://www.percona.com/doc/percona-xtradb-cluster/5.6/upgrading_guide_55_56.html
.. _Galera replication - how to recover a PXC cluster: https://www.percona.com/blog/2014/09/01/galera-replication-how-to-recover-a-pxc-cluster
.. _mysql-innodb-cluster: https://charmhub.io/mysql-innodb-cluster
.. _mysql-router: https://charmhub.io/mysql-router
.. _percona-cluster charm - series upgrade to focal: percona-series-upgrade-to-focal.html
