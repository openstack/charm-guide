=========================================
Adding a charm to the OpenStack ecosystem
=========================================

Overview
--------

After the :doc:`initial implementation <../../concepts/charm-anatomy>` of a new
charm, the charm needs to be added to the following systems in order to be
fully part of our :doc:`release process <../../project/release-schedule>`:

* `Gitea`_, the git server hosting the OpenStack projects.
* `Gerrit`_, the code review platform for contributing to the OpenStack
  projects.
* `Upstream Zuul`_, OpenDev's CI system for the OpenStack projects, publishing
  test results to Gerrit reviews.
* `OSCI`_, Canonical's CI system for the OpenStack charms, also publishing test
  results to Gerrit reviews.
* `Launchpad`_, the bug tracker and continuous delivery pipeline (publishing to
  CharmHub) for the OpenStack charms.
* `Charmed OpenStack Info`_, where the information about Charmed OpenStack is declared.
* `GitHub`_, where the OpenStack charms' source code is mirrored.
* `CharmHub`_, where the charms are published.
* Our `release tools`_ and documentation.

This document will guide you through the necessary steps. If in doubt,
:doc:`contact us <../contact>`.

Initial implementation
----------------------

Put the initial implementation of your new charm in a temporary repository on
GitHub. This repository will be used when doing the initial import to Gitea.
Example: https://github.com/openstack-charmers/charm-nova-compute-nvidia-vgpu

Initial import to Gitea, Gerrit and upstream Zuul
-------------------------------------------------

Follow closely the `OpenDev Project Creator's Guide`_. In a nutshell this will
ask you to:

* Make sure your temporary repository has only one branch and doesn't have a
  ``.zuul.yaml`` file.
* Create a ``project-config`` review. Example:
  https://review.opendev.org/c/openstack/project-config/+/737791
* Create a ``governance`` review. Example:
  https://review.opendev.org/c/openstack/governance/+/737734

Once the import has been performed, retire your original temporary repository.
Example: https://github.com/openstack-charmers/charm-interface-magpie/pull/2

Note that it is possible to import a project with a default git branch named
``main`` instead of ``master``. Example:
https://review.opendev.org/c/openstack/project-config/+/827719/4/gerrit/projects.yaml

Gerrit and upstream Zuul boilerplate
------------------------------------

Add a ``.gitreview`` file and a ``.zuul.yaml`` file to your project. Example:
https://review.opendev.org/c/openstack/charm-neutron-api-plugin-arista/+/738573/

Mirroring to GitHub
-------------------

This is not documented in the `OpenDev Project Creator's Guide`_ but will be at
some point. Create a ``project-config`` review in order to enable the mirroring
from Gitea to GitHub. Example:
https://review.opendev.org/c/openstack/project-config/+/739009

Once this gets merged, a daily job will create the mirror repository at
:samp:`https://github.com/openstack/charm-<my-new-thing>`. Then next time a Gerrit
review gets merged, the initial mirroring will be performed. This process is
still somewhat brittle. If this doesn't work, ask on the #opendev IRC channel
of OFTC.

Launchpad bug tracker
---------------------

Add your project to Launchpad with the following details:

* Part of: ``openstack-charms``
* License: ``Apache``
* Maintainer: ``openstack-charmers``

Example: https://launchpad.net/charm-nova-compute-nvidia-vgpu

Configure the bug tracker at
:samp:`https://bugs.launchpad.net/charm-<my-new-thing>/+configure-bugtracker`
with the following details:

* Expire "Incomplete" bug reports when they become inactive
* Search for possible duplicate bugs when a new bug is filed
* Bug supervisor: ``openstack-charmers``

OSCI
----

Add your project to OSCI's list of known projects. Example:
https://github.com/openstack-charmers/zosci-config/pull/29

Once landed, :doc:`ask us <../contact>` to run the ``reload-config`` action on
our ``zuul-scheduler`` charm.

Add an ``osci.yaml`` file to your project. Example:
https://opendev.org/openstack/charm-aodh/src/branch/master/osci.yaml

If your functional tests require special environment variables in order to run,
add them to OSCI. Example:
https://github.com/openstack-charmers/zosci-config/pull/133

Juju Charm Layers Index
-----------------------

If your new charm is based on the `Reactive framework`_, make sure the
interfaces it requires are listed in the `Juju Charm Layers Index`_. Otherwise
create a pull request. Example:
https://github.com/juju/layer-index/pull/110

Charmed OpenStack Info
----------------------

The `charmed-openstack-info`_ repository contains the definitions for the
OpenStack, Ceph, OVN and supporting charms. Add an entry for the new charm in
the appropriate file under the
``charmed_openstack_info/data/lp-builder-config/`` directory. Example:
https://github.com/openstack-charmers/charmed-openstack-info/pull/23

The format of the ``lp-builder-config`` files is defined in the
`CharmProject`_ class.

Charmhub and Launchpad builders
-------------------------------

Register your charm's name on Charmhub. For example if your repository is named
`charm-<my-new-thing>` do:

.. code-block:: none

   sudo snap install charmcraft --classic
   charmcraft login
   charmcraft register <my-new-thing>

Make sure your charm has a ``charmcraft.yaml`` file so it can be built by the
Launchpad builders. They are responsible for building every commit of your
project and publishing the resulting charm to `Charmhub`_. Example:
https://review.opendev.org/c/openstack/charm-openstack-loadbalancer/+/828162/4/charmcraft.yaml

Add your charm to the `lp-builder-config`_. Example:
https://github.com/openstack-charmers/charmed-openstack-info/pull/23

Once merged, a CI job gets executed which takes care of sync'ing the changes
into Launchpad. Example:
https://github.com/openstack-charmers/charmed-openstack-info/actions/runs/5049412206

Visit :samp:`https://launchpad.net/charm-<my-new-thing>` and for each recipe,
click **Authorize Charmhub uploads**.

Once the ``master`` recipe has succeeded, your charm will be visible at
:samp:`https://charmhub.io/<my-new-thing>`.

Create a `Charmhub request`_ to make ``OpenStack Charmers`` collaborator on your
charm.

Documentation
-------------

Every charm must have a ``README`` file. Construct one by using the :doc:`Charm
README template <charm-readme-template>`.

Add your charm to the project's list of charms and include a release note for
the appropriate OpenStack Charms release. Example:
https://review.opendev.org/c/openstack/charm-guide/+/821962

Add your charm to the upgrade documentation. Example:
https://review.opendev.org/c/openstack/charm-deployment-guide/+/828183

Often the introduction of a new charm coincides with a newly supported feature.
In this case, supporting documentation should be submitted in the form of a
:ref:`management how-to <howto_management>` (or as an addition to an existing
how-to). If the deployment of the new feature is non-trivial then a overlay
bundle that summarises the deployment should be included in the how-to (see
sub-section `Feature overlay bundles`_).

Feature overlay bundles
~~~~~~~~~~~~~~~~~~~~~~~

An overlay bundle for a feature should include placeholder variables that are
intended to be replaced with values that are in accordance with the user's
local environment and/or the intended cloud design. Consider the following
overlay excerpt:

.. code-block:: yaml

   series: $SERIES

   applications:

     ceph-fs:
       charm: ch:ceph-fs
       channel: $CHANNEL_CEPH
       num_units: 2
       options:
         source: $OPENSTACK_ORIGIN

      manila-ganesha:
        charm: ch:ganesha
        channel: $CHANNEL_OPENSTACK
        vip: $VIP

The overlay may optionally include a variables section that makes use of YAML
substitution. For example:

.. code-block:: yaml

   variables:

     data-port: &data-port br-ex:$OVN_DATA_PORT
     osd-devices: &osd-devices $OSD_DEVICES
     network-space: &network-space $SPACE

Wherever the string, for example, ``*network-space`` appears in the overlay it
will be replaced by the value given by ``$SPACE``.

The feature should be deployable during (or after) the deployment of a cloud
- as per the Juju documentation: `How to add an overlay bundle`_.

.. LINKS
.. _charmed-openstack-info: https://github.com/openstack-charmers/charmed-openstack-info
.. _release tools: https://github.com/openstack-charmers/release-tools
.. _Gitea: https://opendev.org/openstack
.. _Gerrit: https://review.opendev.org
.. _Upstream Zuul: https://zuul.openstack.org/status
.. _OSCI: https://wiki.openstack.org/wiki/ThirdPartySystems/Canonical_Charm_CI
.. _Launchpad: https://launchpad.net/~openstack-charmers
.. _GitHub: https://github.com/openstack/
.. _Charmhub: https://charmhub.io/?filter=cloud
.. _Charmhub request: https://discourse.charmhub.io/c/charmhub-requests/46
.. _OpenDev Project Creator's Guide: https://docs.opendev.org/opendev/infra-manual/latest/creators.html
.. _Juju Charm Layers Index: https://github.com/juju/layer-index
.. _lp-builder-config: https://github.com/openstack-charmers/charmed-openstack-info/tree/main/charmed_openstack_info/data/lp-builder-config
.. _charmhub-lp-tools: https://github.com/openstack-charmers/charmhub-lp-tools
.. _Reactive framework: https://charmsreactive.readthedocs.io/en/latest/
.. _Operator framework: https://github.com/canonical/operator
.. _How to add an overlay bundle: https://juju.is/docs/sdk/add-an-overlay-bundle
.. _CharmProject: https://github.com/openstack-charmers/charmhub-lp-tools/blob/main/charmhub_lp_tools/charm_project.py#L47
