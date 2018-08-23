.. _release_notes_<CHARM_RELEASE>

**** This is a template file for future release notes and its contents should not be linked or referenced.  Use actual release notes instead. ****

===========================
<release version goes here>
===========================

Summary
=======

The <CHARM_RELEASE> OpenStack Charm release includes updates for the following charms.  Additional charm support status information is published in the main `charm guide <openstack-charms.html>`__ which ultimately supersedes release note contents.

Supported Charms
~~~~~~~~~~~~~~~~

* aodh
* barbican
* ceilometer
* ceilometer-agent
* ceph-fs
* ceph-mon
* ceph-osd
* ceph-proxy
* ceph-radosgw
* cinder
* cinder-backup
* cinder-ceph
* designate
* designate-bind
* glance
* gnocchi
* hacluster
* heat
* keystone
* keystone-ldap
* lxd
* manila
* manila-generic
* neutron-api
* neutron-openvswitch
* neutron-gateway
* neutron-dynamic-routing
* nova-cloud-controller
* nova-compute
* openstack-dashboard
* percona-cluster
* rabbitmq-server
* swift-proxy
* swift-storage
* vault

Pre-Release Charms
~~~~~~~~~~~~~~~~~~

Deprecated Charms
~~~~~~~~~~~~~~~~~

New Charm Features
==================

OpenStack <O7K_RELEASE> Support
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
The charms provide full support for OpenStack <O7K_RELEASE>. For further details and documentation on Openstack <O7K_RELEASE> please check out https://releases.openstack.org/<O7K_RELEASE>.

Use the 'openstack-origin' charm configuration option to declare the intended OpenStack version, for example:

.. code:: bash

    cat > config.yaml << EOF
    nova-cloud-controller:
      openstack-origin: cloud:bionic-<O7K_RELEASE>
    EOF

    juju deploy --config config.yaml nova-cloud-controller

Also see the published example bundles.

Feature Title 1
~~~~~~~~~~~~~~~

Feature Title 2
~~~~~~~~~~~~~~~


Upgrading charms
================

Please ensure that the keystone charm is upgraded first.

To upgrade an existing deployment to the latest charm version simply use the
'upgrade-charm' command, for example:

.. code:: bash

    juju upgrade-charm keystone

Charm upgrades and OpenStack upgrades are two distinctly different things. Charm upgrades ensure that the deployment is using the latest charm revision, containing the latest charm fixes and charm features available for a given deployment.

Charm upgrades do not cause OpenStack versions to upgrade, however OpenStack upgrades do require the latest Charm version as pre-requisite.

Upgrading OpenStack
===================

To upgrade an existing <O7K_PREVIOUS_RELEASE> based deployment on Ubuntu <UBUNTU_RELEASE> to the <O7K_RELEASE>
release, re-configure the charm with a new openstack-origin
configuration:

.. code:: bash

    juju config nova-cloud-controller openstack-origin=cloud:bionic-<O7K_RELEASE>

Please ensure that ceph services are upgraded before services that consume ceph
resources, such as cinder, glance and nova-compute:

.. code:: bash

    juju config ceph-mon source=cloud:bionic-<O7K_RELEASE>
    juju config ceph-osd source=cloud:bionic-<O7K_RELEASE>

.. note::

   Upgrading an OpenStack cloud is still not without risk; upgrades should
   be tested in pre-production testing environments prior to production deployment
   upgrades.

See https://docs.openstack.org/project-deploy-guide/charm-deployment-guide/latest/app-upgrade-openstack.html for more details.


New Bundle Features
===================

<O7K_RELEASE> Support in Example Bundles
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

<O7K_RELEASE> versions of the example bundles are published in the charm store under cs:openstack-base, cs:openstack-telemetry. The stand-alone ceph bundle is also updated at cs:ceph-base. These bundles have been validated on x86_64, arm64, s390x and ppc64el architectures with Juju 2.3.3 and MAAS 2.3.0.

https://jujucharms.com/openstack-base

https://jujucharms.com/openstack-telemetry

https://jujucharms.com/ceph-base



Deprecation Notices
===================

Notice 1
~~~~~~~~

Notice 2
~~~~~~~~


Known Issues
============

Issue 1
~~~~~~~

Issue 2
~~~~~~~


Bugs Fixed
==========

This release includes NNNN bugs fixes. For the full list of bugs resolved for the <CHARM_RELEASE> charms release please refer to https://launchpad.net/openstack-charms/+milestone/????.

Next Release Info
=================
The next OpenStack Charms release is currently scheduled for ????, and is subject to change.  Please see https://docs.openstack.org/charm-guide/latest for current information.
