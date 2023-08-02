=====================
OpenStack Charm Guide
=====================

The OpenStack Charm Guide is the primary source of documentation for the
OpenStack Charms project. The project oversees the :doc:`collection of Charmed
Operators <project/openstack-charms>`, also called simply "charms", used to
deploy and manage OpenStack clouds using `MAAS`_ and `Juju`_.

.. note::

   This guide supports Juju version 2.9.x. Some breaking changes are introduced
   by Juju 3.x through the renaming of a few commands (see the `Juju 3.0
   release notes`_).

Each of the OpenStack charms is responsible for the deployment and lifecycle
management of a single cloud service (e.g. the nova-compute charm for the Nova
Compute service). The project also includes charms for select non-OpenStack
supporting software, such as Ceph, MySQL, and RabbitMQ.

Deploying and managing an OpenStack cloud is acknowledged among IT
professionals to be a challenging endeavour. The charms reduce the complexity
traditionally imposed upon cloud administrators by doing the heavy lifting.

The OpenStack Charms project is designed to meet the needs of cloud
administrators of varying skill levels. It is appropriate for public, regional,
and private clouds, and can satisfy a wide range of use cases: from small
development environments through to large enterprise-grade solutions.

.. panels::

   Getting started
   ^^^^^^^^^^^^^^^

   Learn how to deploy an OpenStack cloud using Charmed Operators.

   .. toctree::
      :maxdepth: 1

      getting-started/index

   ---

   How-to guides
   ^^^^^^^^^^^^^

   Read step-by-step guides detailing how to work with OpenStack Charmed
   Operators, including key tasks for successful deployment and operations.

   .. toctree::
      :maxdepth: 1

      admin/index

   ---

   Concepts
   ^^^^^^^^

   Dive deep into the concepts of how OpenStack services are set up and managed
   using Charmed Operators.

   .. toctree::
      :maxdepth: 1

      concepts/index

   ---

   Reference
   ^^^^^^^^^

   View the reference material for the OpenStack Charms project.

   .. toctree::
      :maxdepth: 1

      reference/index

Project and community
---------------------

Follow the OpenStack Charms project. This is where you will find a wealth of
information about the project such as the definitive list of OpenStack charms,
release notes, support notes, how to help out, and much more.

.. toctree::
   :maxdepth: 1

   Project <project/index>
   Community <community/index>
   Release notes <release-notes/index>

This project abides by the `OpenInfra Foundation community code of conduct`_.

Search
------

:ref:`Search <search>` this guide.

.. LINKS
.. _MAAS: https://maas.io/
.. _Juju: https://juju.is/
.. _OpenInfra Foundation community code of conduct: https://openinfra.dev/legal/code-of-conduct
.. _Juju 3.0 release notes: https://juju.is/docs/juju/roadmap#heading--juju-3-0-0---22-oct-2022
