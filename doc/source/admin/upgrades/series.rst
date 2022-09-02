==============
Series upgrade
==============

The purpose of this document is to provide foundational knowledge for preparing
an administrator to perform a series upgrade across a Charmed OpenStack cloud.
This translates to upgrading the operating system of every cloud node to an
entirely new version.

Please read the following before continuing:

* :doc:`overview`
* :doc:`../../release-notes/index`
* :doc:`../../project/issues-and-procedures`

Once this document has been studied the administrator will be ready to graduate
to the :doc:`series-openstack` page that describes the process in more detail.

Concerning the cloud being operated upon, the following is assumed:

* It is being upgraded from one LTS series to another (e.g. xenial to
  bionic, bionic to focal, etc.).
* Its nodes are backed by MAAS.
* Its services are highly available.
* It is being upgraded with minimal downtime.

.. warning::

   Upgrading a single production machine from one LTS to another is a serious
   task. Doing so for every cloud node can be that much harder. Attempting to
   do this with minimal cloud downtime is an order of magnitude more complex.

   Such an undertaking should be executed by persons who are intimately
   familiar with Juju and the currently deployed charms (and their related
   applications). It should first be tested on a non-production cloud that
   closely resembles the production environment.

Upgrade candidate availability
------------------------------

Ensure that there is an upgrade candidate available. Charmed OpenStack is
primarily designed to run on Ubuntu LTS releases, and an Ubuntu system is
configured, by default, to upgrade only to the next LTS. In addition, this will
be possible only once the first LTS point release is published (see the `Ubuntu
releases wiki page`_ for release date information). For example, an upgrade to
Focal was possible starting on August 6, 2020.

.. caution::

   The Juju tooling will initiate the upgrade process irrespective of whether
   an upgrade candidate is available or not. A cancelled upgrade is not fatal,
   but it will leave erroneous messaging in :command:`juju status` output.

The Juju :command:`upgrade-series` command
------------------------------------------

The Juju :command:`upgrade-series` command is the cornerstone of the entire
procedure. This command manages an operating system upgrade of a targeted
machine and operates on every application unit hosted on that machine. The
command works in conjunction with either the :command:`prepare` or the
:command:`complete` sub-command.

The basic process is to inform the units on a machine that a series upgrade
is about to commence, to perform the upgrade, and then inform the units that
the upgrade has finished. In most cases with the OpenStack charms, units will
first be paused and be left with a workload status of "blocked" and a message
of "Ready for do-release-upgrade and reboot."

For example, to inform units on machine '0' that an upgrade (to series
'bionic') is about to occur:

.. code-block:: none

   juju upgrade-series 0 prepare bionic

The :command:`prepare` sub-command causes **all** the charms (including
subordinates) on the machine to run their ``pre-series-upgrade`` hook.

The administrator must then perform the traditional steps involved in upgrading
the OS on the targeted machine (in this example, machine '0'). For example,
update/upgrade packages with :command:`apt update && apt full-upgrade`; invoke
the :command:`do-release-upgrade` command; and reboot the machine once
complete.

The :command:`complete` sub-command causes **all** the charms (including
subordinates) on the machine to run their ``post-series-upgrade`` hook. In most
cases with the OpenStack charms, configuration files will be re-written, units
will be resumed automatically (if paused), and be left with a workload status
of "active" and a message of "Unit is ready":

.. code-block:: none

   juju upgrade-series 0 complete

At this point the series upgrade on the machine and its charms is now done. In
the :command:`juju status` output the machine's entry under the Series column
will have changed from 'xenial' to 'bionic'.

.. note::

   Charms are not obliged to support the two series upgrade hooks but they do
   make for a more intelligent and a less error-prone series upgrade.

Containers (and their charms) hosted on the target machine remain unaffected by
this command. However, during the required post-upgrade reboot of the host all
containerised services will naturally be unavailable.

See the Juju documentation to learn more about the `series upgrade`_ feature.

.. _pre-upgrade_requirements:

Pre-upgrade requirements
------------------------

This is a list of requirements that apply to any cloud. They must be met before
making any changes.

* All the cloud nodes should be using the same series, be in good working
  order, and be updated with the latest stable software packages (APT
  upgrades).

* The cloud should be running the latest OpenStack release supported by the
  current series. See `Ubuntu OpenStack release cycle`_ and the
  :doc:`openstack` page.

* The cloud should be fully operational and error-free.

* All currently deployed charms should be upgraded to the latest stable charm
  revision. See the :doc:`charms` page.

* The Juju model comprising the cloud should be error-free (e.g. there should
  be no charm hook errors).

.. _unattended_upgrades:

Unattended upgrades
-------------------

Automatic package updates should be disabled on a node that is about to undergo
a series upgrade. This is to avoid potential conflicts with the manual (or
scripted) APT steps. One way to achieve this is with:

.. code-block:: none

   sudo dpkg-reconfigure -plow unattended-upgrades

Once the upgrade is complete it is advised to re-enable unattended upgrades for
security reasons.

.. _workload_specific_preparations:

Workload specific preparations
------------------------------

These are preparations that are specific to the current cloud deployment.
Completing them in advance is an integral part of the upgrade.

Charm upgradability
~~~~~~~~~~~~~~~~~~~

Verify the documented series upgrade processes for all currently deployed
charms. Some charms, especially third-party charms, may either not have
implemented series upgrade yet or simply may not work with the target series.
Pay particular attention to SDN (software defined networking) and storage
charms as these play a crucial role in cloud operations.

.. _workload_maintenance:

Workload maintenance
~~~~~~~~~~~~~~~~~~~~

Any workload-specific pre and post series upgrade maintenance tasks should be
readied in advance. For example, if a node's workload requires a database then
a pre-upgrade backup plan should be drawn up. Similarly, if a workload requires
settings to be adjusted post-upgrade then those changes should be prepared
ahead of time. Pay particular attention to stateful services due to their
importance in cloud operations. Examples include evacuating a compute node,
switching an HA router to another node, and storage rebalancing.

Pre-upgrade tasks are performed before issuing the :command:`prepare`
subcommand, and post-upgrade tasks are done immediately prior to issuing the
:command:`complete` subcommand.

Workflow: sequential vs. concurrent
-----------------------------------

In terms of the workflow there are two approaches:

* Sequential - upgrading one machine at a time
* Concurrent - upgrading a group of machines simultaneously

Normally, it is best to upgrade sequentially as this ensures data reliability
and availability (we've assumed an HA cloud). This approach also minimises
adverse effects to the deployment if something goes wrong.

However, for even moderately sized clouds, an intervention based purely on a
sequential approach can take a very long time to complete. This is where the
concurrent method becomes attractive.

In general, a concurrent approach is a viable option for API applications but
is not an option for stateful applications. During the course of the cloud-wide
series upgrade a hybrid strategy is a reasonable choice.

To be clear, the above pertains to upgrading the series on machines associated
with a single application. It is also possible however to employ similar
thinking to multiple applications.

Application leadership
----------------------

`Application leadership`_ plays a role in determining the order in which
machines will have their series upgraded. The guiding principle is that an
application's non-leader units (if they exist) are upgraded (in no particular
order) prior to its leader unit. There are exceptions to this however, and they
will be indicated on the :doc:`series-openstack` page.

.. note::

   Juju will not transfer the leadership of an application (and any
   subordinate) to another unit while the application is undergoing a series
   upgrade. This allows a charm to make assumptions that will lead to a more
   reliable outcome.

Assuming that a cloud is intended to eventually undergo a series upgrade, this
guideline will generally influence the cloud's topology. Containerisation is an
effective response to this.

.. important::

   Applications should be co-located on the same machine only if leadership
   plays a negligible role. Applications deployed with the compute and storage
   charms fall into this category.

.. _generic_series_upgrade:

Generic series upgrade
----------------------

This section contains a generic overview of a series upgrade for three
machines, each hosting a unit of the `ubuntu`_ application. The initial and
target series are xenial and bionic, respectively.

This scenario is represented by the following :command:`juju status` command
output:

.. code-block:: console

   Model    Controller       Cloud/Region    Version  SLA          Timestamp
   upgrade  maas-controller  mymaas/default  2.7.6    unsupported  18:33:49Z

   App      Version  Status  Scale  Charm   Store       Rev  OS      Notes
   ubuntu1  16.04    active      3  ubuntu  jujucharms   15  ubuntu

   Unit        Workload  Agent  Machine  Public address  Ports  Message
   ubuntu1/0*  active    idle   0        10.0.0.241             ready
   ubuntu1/1   active    idle   1        10.0.0.242             ready
   ubuntu1/2   active    idle   2        10.0.0.243             ready

   Machine  State    DNS         Inst id  Series  AZ     Message
   0        started  10.0.0.241  node2    xenial  zone3  Deployed
   1        started  10.0.0.242  node3    xenial  zone4  Deployed
   2        started  10.0.0.243  node1    xenial  zone5  Deployed

.. important::

   The asterisk in the Unit column denotes the leader. Here, ``ubuntu1/0`` is
   the leader and its machine ID is 0.

First ensure that any new applications will (by default) use the new series, in
this case bionic. This is done by configuring at the model level:

.. code-block:: none

   juju model-config default-series=bionic

Now do the same at the application level. This will affect any new units of the
existing application, in this case 'ubuntu1':

.. code-block:: none

   juju set-series ubuntu1 bionic

To perform the actual series upgrade we begin with a non-leader machine (1):

.. code-block:: none
   :linenos:

   # Perform any workload maintenance pre-upgrade steps here
   juju upgrade-series 1 prepare bionic
   juju ssh 1 sudo apt update
   juju ssh 1 sudo apt full-upgrade
   juju ssh 1 sudo do-release-upgrade
   # Perform any workload maintenance post-upgrade steps here
   # Reboot the machine (if not already done)
   juju upgrade-series 1 complete

.. note::

   It is recommended to use a terminal multiplexer (e.g. tmux) in order to
   prevent a network disruption from breaking the invoked commands.

In this generic example there are no `workload maintenance`_ steps to perform.
If there were post-upgrade steps then the prompt to reboot the machine at the
end of :command:`do-release-upgrade` should be answered in the negative and the
reboot will be initiated manually on line 7 (i.e. :command:`sudo reboot`).

It is possible to invoke the :command:`complete` sub-command before the
upgraded machine is ready to process it. Juju will block until the unit is
ready after being restarted.

In lines 4 and 5 the upgrade proceeds in the usual interactive fashion. If a
non-interactive mode is preferred, those two lines can be replaced with:

.. code-block:: none

   juju ssh 1 sudo DEBIAN_FRONTEND=noninteractive apt-get --assume-yes \
      -o "Dpkg::Options::=--force-confdef" \
      -o "Dpkg::Options::=--force-confold" dist-upgrade
   juju ssh 1 sudo DEBIAN_FRONTEND=noninteractive \
      do-release-upgrade -f DistUpgradeViewNonInteractive

The :command:`apt-get` command is preferred while in non-interactive mode (or
with scripting).

By default, an LTS release will not have an upgrade candidate until the "point
release" of the next LTS is published. You can override this policy by using
the ``-d`` (development) option with the :command:`do-release-upgrade` command.

.. caution::

   Performing a series upgrade non-interactively can be risky so the decision
   to do so should be made only after careful deliberation.

The remaining non-leader machine (2) is then upgraded:

.. code-block:: none

   juju upgrade-series 2 prepare bionic
   ...
   ...

Finally, the leader machine (0) is upgraded in the same way.

Next steps
----------

When you are ready to perform a series upgrade across your cloud proceed to
the :doc:`series-openstack` page.

.. LINKS
.. _Ubuntu releases wiki page: https://wiki.ubuntu.com/Releases
.. _series upgrade: https://juju.is/docs/olm/upgrade-a-machines-series
.. _Ubuntu OpenStack release cycle: https://ubuntu.com/about/release-cycle#ubuntu-openstack-release-cycle
.. _Application leadership: https://juju.is/docs/olm/leaders
.. _ubuntu: https://charmhub.io/ubuntu
