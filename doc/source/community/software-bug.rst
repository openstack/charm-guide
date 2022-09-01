=========================
Submitting a software bug
=========================

Please read this entire page before submitting an OpenStack Charms bug.
Applying the guidelines given here will considerably reduce the time needed to
triage your bug. Reach out to the team (see :doc:`contact`) if you have any
questions.

Preliminaries
-------------

First consult these resources:

* :doc:`Release notes <../release-notes/index>` - the observed issue may be
  known to the dev team, and a workaround may be available
* `Existing bugs`_ - someone else may have already encountered the issue -
  avoid creating duplicate bugs

The observed issue may already be fixed. Ideally, upgrade to the most recent
revision available and confirm whether the problem persists. See the
:doc:`../admin/upgrades/charms` documentation.

Consider testing for a regression before submitting a bug as it is an extremely
valuable data point. The main variables are Ubuntu series, OpenStack release,
and charm revision/channel.

Cloud stack
~~~~~~~~~~~

Understand the cloud stack to ensure you are filing your bug in the right
place.

Charmed OpenStack is the result of a multi-layered stack, and a problem's root
cause may reside in any one of its layers:

* **charm payload** - OpenStack service (e.g. Nova Compute)
* **charms** - OpenStack charm (e.g. nova-compute)
* **Juju** - modelling software
* **MAAS** - Juju-managed cloud
* **hardware** and **network topology**

The bug you are submitting is for an individual OpenStack charm (the charms
layer).

When `filing the bug`_, select the charm (from the 'Project' dropdown menu)
that you think is most closely related to your problem. For example, if the
problem seems related to a malfunctioning hypervisor, select **OpenStack Nova
Compute Charm** (i.e. the nova-compute charm).

Essential information
---------------------

This section covers the minimal amount of information that every bug should
have.

Please provide:

* an overview of your environment

* a detailed description of the problem and how it differs from your expected
  outcome

* specific software versions in use

  * Ubuntu
  * OpenStack
  * Juju (see `Juju versions`_)
  * MAAS

* if available, concise samples of identified command output errors or log
  messages directly related to the problem

* if known, specific steps that will allow someone else to reproduce the
  problem

See section `Supporting data`_ for details on providing:

* the output to commands :command:`juju status --relations` and :command:`juju export-bundle`
* the logs of any troubled application unit (unit agent logs)

If any of the questions below are answered in the affirmative, provide relevant
details:

* Did the appearance of the problem coincide with a change made to the cloud
  (configuration or topological)?
* Has `AppArmor`_ configuration been modified on the cloud nodes?
* Are :doc:`Policy overrides <../admin/policy-overrides>` in use?

Based on all the above, use a descriptive yet succinct bug title.

Juju versions
-------------

It is possible for the client, the controller model, and the workload model to
each have a different version.

Assuming that the currently active controller is that of the OpenStack cloud,
and the workload model is called 'openstack', you can determine each version as
follows:

The client:

.. code-block:: none

   juju version

The controller model:

.. code-block:: none

   juju show-model controller | grep agent-version

The workload model:

.. code-block:: none

   juju show-model openstack | grep agent-version

Supporting data
---------------

When providing supporting data, use a separate file for each different type of
data and attach them to the bug. Include a summary of what each file contains
(in the bug description) if you think it will facilitate bug triage.

Avoid using a third-party service (e.g. pastebin or imagebin) as data hosted in
this way is not considered permanent.

The below sections cover the most common types of supporting data.

.. contents::
   :local:
   :depth: 2
   :backlinks: none

CLI commands
~~~~~~~~~~~~

``status``
^^^^^^^^^^

The :command:`juju status` command is a staple when communicating the state of
a model. Here we also include the relations:

.. code-block:: none

   juju status --relations > juju-status_relations.txt

``export-bundle``
^^^^^^^^^^^^^^^^^

The :command:`juju export-bundle` command inspects a model and generates a
bundle file from it. This will give a good understanding as to how the cloud
was deployed.

.. code-block:: none

   juju export-bundle --filename juju-export-bundle.txt

``config``
^^^^^^^^^^

The :command:`juju config` command retrieves a charm's configuration options
and their corresponding current values. These options alter how the charm and
its payload behave together. Not only does this information help in
understanding the environment but it will also reveal an incorrectly set
option.

To retrieve the configuration for a charm (ceph-osd here):

.. code-block:: none

   juju config ceph-osd > juju-config_ceph-osd.txt

``crashdump``
^^^^^^^^^^^^^

The :command:`juju crashdump` command generates a comprehensive, yet
**unsanitised**, report on an entire Juju model. It is available via a Juju
plugin. Install it alongside the Juju client:

.. code-block:: none

   sudo snap install juju-crashdump --classic

For example, to analyse the currently active model and tag the report with a
unique string (assuming the issue involves the ovn-central charm):

.. code-block:: none

   juju crashdump --small --as-root -o ~/tmp -u ovn-central

This will produce the file ``~/tmp/juju-crashdump-ovn-central.tar.xz``.

Omitting the ``--small`` option will lead to the inclusion of a massive amount
of Juju debug information (see `Dealing with large file attachments`_). To get
more command help: ``juju crashdump --help``.

Omitting the ``--as-root`` option will prevent certain logs (and effectively
more sensitive information) from being collected.

.. note::

   To avoid copying the file across networks in order to attach it to the bug
   (the file is probably not immediately available to your browser), the
   command's ``-b`` option can be used to send it directly to an existing bug.

Logs
~~~~

Logs are often an essential type of supporting data. With Charmed OpenStack
there are two main categories: Juju agent logs and OpenStack service logs.

Agent logs
^^^^^^^^^^

Juju agents record events from the perspective of Juju. Unit agents are
generally more useful in the present context as they interface with the payload
(OpenStack) whereas machine agents are concerned with the provisioning of the
Juju machine.

To retrieve unit agent logs (``nova-compute/0`` here):

.. code-block:: none

   juju debug-log --replay --no-tail --include nova-compute/0 | tee bug_1234567_nova-compute_0.log

To retrieve machine agent logs (``machine-8`` here):

.. code-block:: none

   juju debug-log --replay --no-tail --include machine-8 | tee bug_1234567_machine_8.log

Create an archive (e.g. with the :command:`tar` command) of the desired logs
and attach it to the bug.

You can set the logging verbosity (of the currently active model) for both
types of agents. Here we set the level of both the machine agent (``<root>``)
and the unit agent (``unit``) to 'DEBUG':

.. code-block:: none

   juju model-config logging-config="<root>=DEBUG;unit=DEBUG"

The `Juju logs`_ page in the Juju documentation has more details.

Service logs
^^^^^^^^^^^^

Service logs are the native logs of the OpenStack service in question. They are
found in their standard locations under ``/var/log`` on each individual
machine. Create an archive (e.g. with the :command:`tar` command) of the
desired logs and attach it to the bug.

To increase the verbosity of these logs for an application (nova-compute here):

.. code-block:: none

   juju config nova-compute debug=true

CLI session
~~~~~~~~~~~

A CLI session is a series of terminal-based commands and their respective
outputs. This is very useful in conveying an exact chronology of what was
done/attempted and what the results were.

Screenshots
~~~~~~~~~~~

Screenshots are typically used when the subject is graphical in nature such as
the web UIs available with MAAS, OpenStack Horizon, and Ceph Dashboard.

Dealing with large file attachments
-----------------------------------

Attaching an oversized file to the bug can be problematic (Launchpad may time
out). In such cases, the common :command:`split` utility can be of use.
Consider the below :command:`juju-crashdump` report:

.. code-block:: console

   -rw-rw-r-- 1 ubuntu ubuntu 167M Feb  7 22:06 juju-crashdump-7c9c30a8-686c-4d28-8765-b31c1791ca85.tar.xz

To break it into 64MiB chunks (and add some prefix and suffix information to
the resulting files):

.. code-block:: none

   split -b 64M --numeric-suffixes=1 --additional-suffix=-juju-crashdump \
      juju-crashdump-7c9c30a8-686c-4d28-8765-b31c1791ca85.tar.xz split-

This yields three manageable files:

.. code-block:: console

   -rw-rw-r-- 1 ubuntu ubuntu  64M Feb  8 16:32 split-01-juju-crashdump
   -rw-rw-r-- 1 ubuntu ubuntu  64M Feb  8 16:32 split-02-juju-crashdump
   -rw-rw-r-- 1 ubuntu ubuntu  39M Feb  8 16:32 split-03-juju-crashdump

Please include an explanatory bug comment:

::

   I have split a juju-crashdump file into three and attached them. To
   reconstruct:

   $ cat split-0?-juju-crashdump > juju-crashdump.tar.xz

.. LINKS
.. _Juju logs: https://juju.is/docs/olm/juju-logs
.. _AppArmor: https://ubuntu.com/server/docs/security-apparmor
.. _Existing bugs: https://bugs.launchpad.net/openstack-charms/+bugs?orderby=-id&start=0
.. _filing the bug: https://bugs.launchpad.net/openstack-charms/+filebug
