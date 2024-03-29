====
Yoga
====

The Yoga OpenStack Charms release includes updates for the charms described on
the :doc:`../project/openstack-charms` page. As of this release, the project
consists of 62 stable charms.

For the list of bugs resolved in this release refer to the `22.04 milestone`_
in Launchpad.

For scheduling information of past and future releases see the
:doc:`../project/release-schedule`.

.. note::

   Release Notes contents is superseded by updated information published in the
   `OpenStack Charm Guide`_ (this guide) after the release of any given
   OpenStack Charms version.

.. important::

   Always upgrade to the latest stable charms before making any major changes
   to your cloud and before filing bug reports. Note that charm upgrades and
   OpenStack upgrades are functionally different. For instructions on
   performing the different upgrade types see `Upgrades overview`_ in the
   `OpenStack Charms Deployment Guide`_.

.. contents:: Summary of changes:
   :local:
   :depth: 2
   :backlinks: top

New stable charm features
-------------------------

With each new feature, there is a corresponding example bundle in the form of a
test bundle, and/or a section in the documentation that details its usage. Test
bundles are located in the ``src/tests/bundles`` directory of the relevant
charm repository (see all `charm repositories`_).

nova-compute charm: vTPM support
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The nova-compute charm now supports virtual TPM (vTPM) devices and is enabled
via a new configuration option: ``enable-vtpm``. When it is set to 'True', the
Nova and libvirt services will be configured to provide vTPM devices via swtpm.

This requires the swtpm, swtpm-tools, and libtpm0 packages to be available for
installation on the Compute nodes. These are available in the Ubuntu 22.04 LTS
release and are expected to be backported to Ubuntu 20.04 LTS. In the interim,
the OpenStack Charmers team has included a backport of these packages in the
``ppa:openstack-charmers/swtpm`` PPA.

glance charm: Cinder iSCSI/FC backend storage support
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The glance charm adds support for Cinder iSCSI/FC backend storage. This is
managed through three new configuration options:

``cinder-volume-types``
  A comma-separated list of Cinder volume_types that can be configured as
  Cinder storage backends. The first one in the list will be configured as the
  default backend.

``cinder-http-retries``
  The number of cinderclient retries on failed HTTP calls.

``cinder-state-transition-timeout``
  The timeout (seconds) of a Cinder volume transition.

keystone charm: new action to query Keystone admin password
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The keystone charm has a new ``get-admin-password`` action. This action
retrieves the admin password of the Keystone service.

cinder charm: new option to expose scheduler_default_filters
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The cinder charm has a new configuration option: ``scheduler-default-filters``.
It sets the value of ``scheduler_default_filters`` in the Cinder configuration.

ironic-conductor charm: new option to expose use_ipmitool_retries
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The ironic-conductor charm has a new configuration option:
``use-ipmitool-retries``. This option sets the value of
``use_ipmitool_retries`` in the Ironic configuration.

ceph-mon charm: add list-crush-rules action
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The ceph-mon charm has a new ``list-crush-rules`` action, which provides a list
of CRUSH rules defined in Ceph clusters.

The action has a ``format`` parameter that accepts the following values:

* 'json' - provides detailed information in json format
* 'yaml' - provides detailed information in yaml format
* 'text' - provides less information in human readable format [default]

nova-cloud-controller charm: new option to set scheduling retries
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The nova-cloud-controller charm has a new configuration option:
``scheduler-max-attempts``. This will allow for an increase in the number of
retries and hence hosts to schedule on, thereby helping in a successful
scheduling of instances. It sets the scheduler.max attempts in the Nova
configuration.

nova-compute charm: new option to add extra repositories
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The nova-compute charm has a new configuration option for adding extra apt
repositories to Compute nodes: ``extra-repositories``. This option takes a
comma-delimited list of apt source repository spec entries to add as apt
package repositories. The valid values are those accepted by the
:command:`add-apt-repository` command.

neutron-api charm: new option to enable PF link status propagation to VF
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The neutron-api charm has a new configuration option for allowing the PF
(physical function) link status for an OpenStack SR-IOV port on the host to be
propagated to the VF (virtual function) link status on a cloud instance. To
enable this, set the option to 'true' and assign attribute
'propagate_uplink_status' to the SR-IOV port during its creation (via the flag
``--enable-uplink-status-propagation``).

.. note::

   This feature is available starting with OpenStack Stein.

Increase KeepAlive timeout for OpenStack API endpoints and dashboard
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The HTTP KeepAlive timeout for OpenStack API endpoints and the Dashboard
has been changed to 75 seconds. The previous timeout of 5 seconds
(Apache's default) was causing unnecessary termination of client TCP
connections, which was also affecting inter-service (OpenStack)
communication.

neutron-api charm: new option to enable network-segment-range support
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The neutron-api charm has a new option for enabling Neutron's 'Network
Segment Range' service plugin: ``enable-network-segment-range``. It
allows cloud operators to dynamically manage network segment ranges
through the Neutron API. For more details, refer to `Network segment
ranges`_ in the upstream documentation.

nova-compute charm: new options to set block device timeout
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The nova-compute charm has two new options:

* ``block-device-allocate-retries``
* ``block-device-allocate-retries-interval``

These options configure the block device allocation timeout. The default
values have been set at 300 and 3 respectively, resulting in an overall
timeout of 15 minutes. The previous (inherited upstream) timeout of 3
minutes resulted in failures when dealing with large guest images (e.g.
Windows or customised Linux).

nova-cloud-controller charm: new option to expose max_local_block_devices
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The nova-cloud-controller charm has a new option:
``max-local-block-devices``. It exposes the upstream
max_local_block_devices flag. In particular, setting it to '0' will
forbid local block devices, effectively compelling users to request
volumes instead. For more information, refer to `block device mapping
FAQ section`_ in the upstream documentation.

nova-cloud-controller charm: new options to filter hosts by isolating aggregates
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The nova-cloud-controller charm has three new options:

* ``limit-tenants-to-placement-aggregate``
* ``placement-aggregate-required-for-tenants``
* ``enable-isolated-aggregate-filtering``

These options are useful for limiting host aggregates to specific
tenants. For more information, refer to `Filtering hosts by isolating
aggregates`_ in the upstream documentation.

nova-compute charm: new option to expose reserved_host_disk_mb
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The nova-compute charm has a new option: ``reserved-host-disk``. It
takes into account available host disk space when scheduling instances.
It is similar to existing options ``reserved-host-memory`` and
``reserved-huge-pages``.

openstack-dashboard charm: new options to allow for branding
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The openstack-dashboard charm has three new options:

* ``site-branding``
* ``site-branding-link``
* ``help-url``

These options are used to set Dashboard parameters that reflect the
local environment.

ceph-osd: new options for disk creation
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The ``add-disk`` action for the ``ceph-osd`` charm has incorporated the
following options:

* ``osd-ids``
* ``cache-devices``
* ``partition-size``

ceph-osd: new action to remove disks
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The ceph-osd charm has a new ``remove-disk`` action. This action allows
operator to remove previously created disks.

Documentation updates
---------------------

A summary of the most significant documentation updates is given below.

In the `OpenStack Charm Guide`_:

* More cloud operations
* Improvements to the upgrade pages
* New tutorial for deploying OpenStack
* New guidelines and resources for documentation and software contributions
* Add a spellchecker to the build process
* New page on virtual TPM devices
* Refactor of the Project and Community sections

New tech-preview charms
-----------------------

ceph-nfs
~~~~~~~~

The ceph-nfs charm provides action-managed NFS storage backed by CephFS. It is
a principal charm used in conjunction with a deployed Ceph cluster.

cinder-nimblestorage
~~~~~~~~~~~~~~~~~~~~

The cinder-nimblestorage charm provides NimbleStorage storage backend support
for the OpenStack Cinder service. It is a subordinate charm used in conjunction
with the cinder principal charm.

cinder-solidfire
~~~~~~~~~~~~~~~~

The cinder-solidfire charm provides SolidFire storage backend support for
the OpenStack Cinder service. It is a subordinate charm used in conjunction
with the cinder principal charm.

nova-compute-nvidia-vgpu
~~~~~~~~~~~~~~~~~~~~~~~~

The nova-compute-nvidia-vgpu charm provides Nvidia vGPU support to the
OpenStack Nova Compute service. It is a subordinate charm used in conjunction
with the nova-compute principal charm.

Informational notices
---------------------

nova-cloud-controller: cpu-allocation-ratio default changed to 2
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The default value for cpu-allocation-ratio has been reduced from 16 to 2. The
old default was more appropriate for dev, test, or lab type environments but is
rarely suitable for clouds running production workloads. If you were relying on
the previous default of 16 and start to see VM scheduling failures after the
upgrade of this charm, you can opt back into a higher contention ratio by
running:

.. code-block:: none

   juju config nova-cloud-controller cpu-allocation-ratio=16

rabbitmq-server: options removed
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The configuration options ``modulo-nodes`` and ``known-wait`` have been removed
from the rabbitmq-server charm as development work related to coordinated node
restarts has made them irrelevant.

Issues discovered during this release cycle
-------------------------------------------

glance-simplestreams-sync: endpoint change
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The ceph-radosgw charm improves how URLs are processed by the RADOS Gateway.
This change however will lead to breakage for an existing ``product-streams``
endpoint, set up by the glance-simplestreams-sync application. Manual
intervention is required - see the :ref:`Upgrade issues
<charm_upgrade_issue-radosgw_gss>` page for more information.

Nova fails to parse new libvirt mediated device name format
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The name format of mediated devices in libvirt was recently changed from
``mdev_<uuid>`` to ``mdev_<uuid>_<parent>``. For users of the new tech-preview
nova-compute-nvidia-vgpu charm, this will cause new Nova instances to enter
into an error state subsequent to a vGPU device being attached to an instance.
This is being tracked in issue `LP #1951656`_. A fix will soon be available as
an SRU.

glance-simplestreams-sync: Juju resources support and snap install
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The glance-simplestreams-sync charm has gained support for Juju resources.
Since Simplestreams is now installed via a snap, offline environments can
benefit by being able to supply the snap as a resource.

.. LINKS
.. _22.04 milestone: https://launchpad.net/openstack-charms/+milestone/22.04
.. _OpenStack Charms Deployment Guide: https://docs.openstack.org/project-deploy-guide/charm-deployment-guide/latest
.. _OpenStack Charm Guide: https://docs.openstack.org/charm-guide/latest/
.. _Upgrades overview: https://docs.openstack.org/charm-guide/latest/admin/upgrades/overview.html
.. _charm repositories: https://opendev.org/openstack?sort=alphabetically&q=charm-&tab=
.. _Network segment ranges: https://docs.openstack.org/neutron/latest/admin/config-network-segment-ranges.html
.. _block device mapping FAQ section: https://docs.openstack.org/nova/latest/user/block-device-mapping.html#faqs
.. _Filtering hosts by isolating aggregates: https://docs.openstack.org/nova/latest/reference/isolate-aggregates.html

.. COMMITS

.. BUGS
.. _LP #1951656: https://bugs.launchpad.net/nova/+bug/1951656
