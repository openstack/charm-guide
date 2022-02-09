:orphan:

=====================
Charm README template
=====================

Overview
--------

The purpose of the README template is to help in providing consistency across
the collection of charms. It also helps to reduce the amount of effort needed
during the commit review phase (for both author and reviewer) when new charms
are developed.

The writing format is Markdown, which can be validated using a `Markdown
viewer`_. The README also gets rendered on the charm's landing page in the
`Charmhub`_ (with the `Mistune`_ Python parser).

Please see the :doc:`Writing style guide <doc-style-guide>` for the OpenStack
Charms project.

General approach
----------------

The README should encapsulate the purpose of a charm and its basic usage. It
should be streamlined yet informative enough to allow a user to deploy the
charm and to benefit from the charm's most common use case.

Any surplus documentation should be submitted to the `OpenStack Charm Guide`_
and then linked to (i.e. "For more information see..."). :doc:`Ask <contact>`
for advice if you're unsure about how to submit this sort of documentation.

When referencing another charm in a general way, link to the charm's Charmhub
entry. When referring specifically to information in another charm's README,
link directly to the file, or to a header in the file, by way of the charm's
repository on `opendev.org`_.

Structure
---------

The file's structure is given below in terms of section headers. The sections
in bold are required. Any other sections must be included if they apply to the
charm.

**# Overview**

**# Configuration**

**# Deployment**

# Actions

# <charm feature X>

# <charm feature Y>

# <charm feature Z>

# High availability

# Policy overrides

# Deferred service events

**# Documentation**

**# Bugs**

Section **Overview**
~~~~~~~~~~~~~~~~~~~~

Always include this section.

Typically the charm is associated with an upstream project - often, but not
always, an OpenStack project. Very briefly give the purpose of that project's
service.

Directly state what the charm does and in what context it is deployed. For
instance, if it works in tandem with another charm then state that.

Use an admonishment to indicate whether the charm is in a tech-preview state
and/or whether the charm has requirements/limitations in terms of an OpenStack
release or an Ubuntu series.

Section **Configuration**
~~~~~~~~~~~~~~~~~~~~~~~~~

Always include this section.

This is boilerplate text:

.. code-block:: none

   # Configuration

   To display all configuration option information run `juju config
   <application>`. If the application is not deployed then see the charm's
   [Configure tab][<charm>-configure] in the Charmhub. Finally, the [Juju
   documentation][juju-docs-config-apps] provides general guidance on
   configuring applications.

Fill in the placeholder for the charm's Charmhub entry.

.. important::

   There is no need to call out specific configuration options in the README.
   However, make sure that the ``config.yaml`` file explains each option well.

Section **Deployment**
~~~~~~~~~~~~~~~~~~~~~~

Always include this section.

The user should be able to deploy the charm for the most common use cases by
following the instructions given in this section.

Although an application typically never exists in isolation, strive to avoid
duplicating deployment instructions for other charms. Instead, call out those
applications that the new application requires a relation to. Then provide all
the commands necessary to make the associated model "go green" (no unsatisfied
relations or configurations). The `nova-cloud-controller`_ README is a good
example.

Give any general requirements. For example, whether a pre-existing Ceph cluster
is assumed.

As exceptions to the above rule, subordinate charms can include the deployment
steps of a principal charm (e.g. `cinder-ceph`_ and cinder). Also, some charms
are so closely related that it makes sense for each to show both (e.g.
`swift-proxy`_ and `swift-storage`_, or `ceph-mon`_ and `ceph-osd`_),
especially if the space required is minimal. Use common sense.

Section **Actions**
~~~~~~~~~~~~~~~~~~~

Include this section if it applies to the charm.

This is boilerplate text:

.. code-block:: none

   # Actions

   This charm supports actions.

   [Actions][juju-docs-actions] allow specific operations to be performed on a
   per-unit basis. To display actions and their descriptions run `juju actions
   --schema <application>`. If the application is not deployed then see the
   charm's [Actions tab][<charm>-actions] in the Charmhub.

Fill in the placeholder for the charm's Charmhub entry.

.. important::

   There is no need to call out specific actions in the README.  However, make
   sure that the ``actions.yaml`` file explains each action well.

Section **<charm feature>**
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Include a section for each noteworthy feature the charm may have.

Section **High availability**
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Include this section if it applies to the charm.

Most services support some form of high availability. When one does, it is
either natively HA or non-natively HA (requires HAcluster). Include text for a
charm's HA implementation.

This is boilerplate text for a non-native HA service:

.. code-block:: none

   # High availability

   This charm supports high availability via HAcluster.

   When more than one unit is deployed with the [hacluster][hacluster-charm]
   application the charm will bring up an HA active/active cluster.

See the `rabbitmq-server`_ charm for an example of a native HA service.

Regardless of the nature of the charm's HA implementation, the section should
always include this boilerplate text, and :doc:`Alert <contact>` the team if
your charm is not conceptually covered in the specified resource:

.. code-block:: none

   See [Infrastructure high availability][cg-ha-apps] for more information.

Section **Policy overrides**
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Include this section if it applies to the charm.

This is boilerplate text:

.. code-block:: none

   # Policy overrides

   This charm supports the policy overrides feature.

   Policy overrides is a feature that allows an operator to override the
   default policy of an OpenStack service.

   See [Policy overrides][cg-policy-overrides] for more information on this
   feature.

Section **Deferred service events**
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Include this section if it applies to the charm.

This is boilerplate text:

.. code-block:: none

   # Deferred service events

   This charm supports the deferred service events feature.

   Operational or maintenance procedures applied to a cloud often lead to the
   restarting of various OpenStack services and/or the calling of certain charm
   hooks. Although normal, such events can be undesirable due to the service
   interruptions they can cause.

   The deferred service events feature provides the operator the choice of
   preventing these service restarts and hook calls from occurring, which can
   then be resolved at a more opportune time.

   See [Deferred service events][cg-deferred-service-events] for more
   information on this feature.

Section **Documentation**
~~~~~~~~~~~~~~~~~~~~~~~~~

Always include this section.

This is boilerplate text:

.. code-block:: none

   # Documentation

   The OpenStack Charms project maintains two documentation guides:                                                                                             

   * [OpenStack Charm Guide][cg]: the primary source of information for
     OpenStack charms
   * [OpenStack Charms Deployment Guide][cdg]: a step-by-step guide for
     deploying OpenStack with charms

Section **Bugs**
~~~~~~~~~~~~~~~~

Always include this section.

This is boilerplate text:

.. code-block:: none

   # Bugs

   Please report bugs on [Launchpad][<charm>-filebug].

Fill in the placeholder for the charm's bug-filing link.

Links
-----

Put all links at the bottom. For example:

.. code-block:: none

   <!-- LINKS -->

   [cg]: https://docs.openstack.org/charm-guide
   [cg-deferred-service-events]: https://docs.openstack.org/charm-guide/latest/admin/deferred-events.html
   [cg-policy-overrides]: https://docs.openstack.org/charm-guide/latest/admin/policy-overrides.html
   [cg-ha-apps]: https://docs.openstack.org/project-deploy-guide/charm-deployment-guide/latest/ha.html#ha-applications
   [cdg]: https://docs.openstack.org/project-deploy-guide/charm-deployment-guide
   [hacluster-charm]: https://charmhub.io/hacluster
   [juju-docs-actions]: https://juju.is/docs/working-with-actions
   [juju-docs-config-apps]: https://juju.is/docs/configuring-applications
   [<charm>-actions]: https://charmhub.io/<charm>/actions
   [<charm>-configure]: https://charmhub.io/<charm>/configure
   [<charm>-filebug]: https://bugs.launchpad.net/charm-<charm>/+filebug

.. LINKS
.. _Charmhub: https://charmhub.io
.. _opendev.org: https://opendev.org/explore/repos?tab=&sort=recentupdate&q=charm-
.. _Markdown viewer: https://jbt.github.io/markdown-editor
.. _Mistune: https://mistune.readthedocs.io/en/latest
.. _OpenStack Charm Guide: https://docs.openstack.org/charm-guide
.. _rabbitmq-server: https://opendev.org/openstack/charm-rabbitmq-server/src/branch/master/README.md#high-availability
.. _swift-proxy: https://opendev.org/openstack/charm-swift-proxy/src/branch/master/README.md
.. _swift-storage: https://opendev.org/openstack/charm-swift-storage/src/branch/master/README.md
.. _nova-cloud-controller: https://opendev.org/openstack/charm-nova-cloud-controller/src/branch/master/README.md
.. _cinder-ceph: https://opendev.org/openstack/charm-cinder-ceph/src/branch/master/README.md
.. _ceph-mon: https://opendev.org/openstack/charm-ceph-mon/src/branch/master/README.md
.. _ceph-osd: https://opendev.org/openstack/charm-ceph-osd/src/branch/master/README.md
