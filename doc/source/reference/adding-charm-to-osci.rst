================================================
Integrating a charm into the OpenStack ecosystem
================================================

Overview
--------

After the `initial implementation`_ of a new charm, the charm needs to be added
to the following systems in order to be fully part of our `release process`_:

* `Gitea`_, the git server hosting the OpenStack projects.
* `Gerrit`_, the code review platform for contributing to the OpenStack
  projects.
* `Upstream Zuul`_, OpenDev's CI system for the OpenStack projects, publishing
  test results to Gerrit reviews.
* `OSCI`_, Canonical's CI system for the OpenStack charms, also publishing test
  results to Gerrit reviews.
* `Launchpad`_, the bug tracker and continuous delivery pipeline (publishing to
  CharmHub) for the OpenStack charms.
* `GitHub`_, where the OpenStack charms' source code is mirrored.
* `CharmHub`_, where the charms are published.
* Our `release tools`_ and documentation.

This document will guide you through all the necessary steps. Please don't skip
any step. If in doubt, :doc:`contact us <../community/contact>`.


Initial implementation
----------------------

Put the `initial implementation`_ of your new charm in a temporary repository
on GitHub. This repository will be used when doing the initial import to Gitea.
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

Update your charm's ``README`` file. Example:
https://review.opendev.org/c/openstack/charm-neutron-api-plugin-arista/+/739467/


OSCI
----

Add your project to OSCI's list of known projects. Example:
https://github.com/openstack-charmers/zosci-config/pull/29

Once landed, :doc:`ask us <../community/contact>` to run the
``reload-config`` action on our ``zuul-scheduler`` charm.

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


Release Tools
-------------

The ``release-tools`` repository contains lists of all OpenStack charms:

* ``operator-charms.txt``: add your charm to this list if it has been
  implemented using the `Operator framework`_. Example:
  https://github.com/openstack-charmers/release-tools/pull/176
* ``charms.txt`` and ``source-charms.txt``: add your charm to these lists if it
  has been implemented using the `Reactive framework`_. Example:
  https://github.com/openstack-charmers/release-tools/pull/119


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
https://github.com/openstack-charmers/release-tools/pull/187

Once landed, :doc:`ask us <../community/contact>` to run the
`charmhub-lp-tools`_ in order to create the corresponding Launchpad builder
recipes:

.. code-block:: none

   charmhub-lp-tool sync --i-really-mean-it

Visit :samp:`https://launchpad.net/charm-<my-new-thing>` and for each recipe,
click **Authorize Charmhub uploads**.

Once the ``master`` recipe has succeeded, your charm will be visible at
:samp:`https://charmhub.io/<my-new-thing>`.

Create a `Charmhub request`_ to make ``OpenStack Charmers`` collaborator on your
charm.


Documentation
-------------

Advertise your new charm to the charm-guide and its `release notes`_. Example:
https://review.opendev.org/c/openstack/charm-guide/+/821962

Add your new charm to the charm-deployment-guide and its upgrade documentation.
Example: https://review.opendev.org/c/openstack/charm-deployment-guide/+/828183

If your charm has in-depth documentation consider adding a page to the
charm-deployment-guide (and linking to it from its ``README``).


.. LINKS
.. _initial implementation: charm-anatomy.html
.. _release process: release-schedule.html
.. _release notes: ../release-notes/index.html
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
.. _lp-builder-config: https://github.com/openstack-charmers/release-tools/tree/master/lp-builder-config
.. _charmhub-lp-tools: https://github.com/openstack-charmers/charmhub-lp-tools
.. _Reactive framework: https://charmsreactive.readthedocs.io/en/latest/
.. _Operator framework: https://github.com/canonical/operator
