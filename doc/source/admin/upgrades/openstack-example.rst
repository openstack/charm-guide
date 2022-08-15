:orphan:

=========================
OpenStack upgrade example
=========================

This document shows the specific steps used to perform an OpenStack upgrade.
They are based entirely on the :doc:`openstack` page.

Cloud description
-----------------

The original cloud deployment was performed via this 'focal-wallaby'
``openstack-base`` `bundle`_. The backing cloud is a MAAS cluster consisting of
three physical machines.

In order to demonstrate upgrading an application running under HA with the aid
of the hacluster subordinate application, Keystone was scaled out post-deploy:

.. code-block:: none

   juju add-unit -n 2 --to lxd:1,lxd:2 keystone
   juju config keystone vip=10.246.116.11
   juju deploy --config cluster_count=3 hacluster keystone-hacluster
   juju add-relation keystone-hacluster:ha keystone:ha

The pre-upgrade state and topology of the cloud is given via this :doc:`model
status output <openstack-example-pre-juju-status>`.

Objective
---------

Since the cloud was deployed with a UCA OpenStack release of 'focal-wallaby',
the upgrade target is 'focal-xena'. The approach taken is one that minimises
service downtime while the upgrade is in progress.

Prepare for the upgrade
-----------------------

It is assumed that the :ref:`preparatory steps <openstack_upgrade_prepare>`
have been completed.

Perform the upgrade
-------------------

Perform the upgrade by following the below sections.

Disable unattended-upgrades
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Disable unattended-upgrades on the three cloud nodes. Recall that each one
directly hosts multiple applications (e.g. ceph-osd, cinder, and nova-compute
are deployed on machine 2):

.. code-block:: none

   juju ssh 0 sudo dpkg-reconfigure -plow unattended-upgrades
   juju ssh 1 sudo dpkg-reconfigure -plow unattended-upgrades
   juju ssh 2 sudo dpkg-reconfigure -plow unattended-upgrades

Answer 'No' to the resulting question.

Perform a backup of the service databases
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Determine the existing service databases and then back them up.

.. code-block:: none

   PASSWORD=$(juju run -u mysql-innodb-cluster/leader leader-get mysql.passwd)
   juju ssh mysql-innodb-cluster/leader "mysql -u root -p${PASSWORD} -e 'SHOW DATABASES;'"

   +-------------------------------+
   | Database                      |
   +-------------------------------+
   | cinder                        |
   | glance                        |
   | horizon                       |
   | information_schema            |
   | keystone                      |
   | mysql                         |
   | mysql_innodb_cluster_metadata |
   | neutron                       |
   | nova                          |
   | nova_api                      |
   | nova_cell0                    |
   | performance_schema            |
   | placement                     |
   | sys                           |
   | vault                         |
   +-------------------------------+

By omitting the system databases we are left with:

* ``cinder``
* ``glance``
* ``horizon``
* ``keystone``
* ``neutron``
* ``nova``
* ``nova_api``
* ``nova_cell0``
* ``placement``
* ``vault``

Now run the following commands:

.. code-block:: none

   juju run-action --wait mysql-innodb-cluster/0 mysqldump \
      databases=cinder,glance,horizon,keystone,neutron,nova,nova_api,nova_cell0,placement,vault
   juju run -u mysql-innodb-cluster/0 -- sudo chmod o+rx /var/backups/mysql
   juju scp -- -r mysql-innodb-cluster/0:/var/backups/mysql .
   juju run -u mysql-innodb-cluster/0 -- sudo chmod o-rx /var/backups/mysql

Move the transferred archive to a safe location (off of the client host).

Archive old database data
~~~~~~~~~~~~~~~~~~~~~~~~~

Archive old database data by running an action on any nova-cloud-controller
unit:

.. code-block:: none

   juju run-action --wait nova-cloud-controller/0 archive-data

Repeat this command until the action output reports 'Nothing was archived'.

Purge old compute service entries
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Purge any old compute service entries for nova-compute units that are no longer
part of the model. These entries will show as 'down' in the list of compute
services:

.. code-block:: none

   openstack compute service list

To remove a compute service:

.. code-block:: none

   openstack compute service delete <service-id>

List the upgrade order
~~~~~~~~~~~~~~~~~~~~~~

From an excerpt of the initial :command:`juju status` output, create an
inventory of running applications:

.. code-block:: console

   ceph-mon
   ceph-osd
   ceph-radosgw
   cinder
   cinder-ceph
   cinder-mysql-router
   dashboard-mysql-router
   glance
   glance-mysql-router
   keystone
   keystone-mysql-router
   mysql-innodb-cluster
   neutron-api
   neutron-api-plugin-ovn
   neutron-mysql-router
   nova-cloud-controller
   nova-compute
   nova-mysql-router
   ntp
   openstack-dashboard
   ovn-central
   ovn-chassis
   placement
   placement-mysql-router
   rabbitmq-server
   vault
   vault-mysql-router

Ignore from the above all subordinate applications and those applications that
are not part of the UCA. After applying the recommended upgrade order we arrive
at the following ordered list:

#. ceph-mon
#. keystone
#. ceph-radosgw
#. cinder
#. glance
#. neutron-api
#. ovn-central
#. placement
#. nova-cloud-controller
#. openstack-dashboard
#. nova-compute
#. ceph-osd

Upgrade each application
~~~~~~~~~~~~~~~~~~~~~~~~

Upgrade each application in turn.

ceph-mon
^^^^^^^^

Although there are three units of the ceph-mon application, the all-in-one
method is used because the ceph-mon charm is able to maintain service
availability during the upgrade:

.. code-block:: none

   juju config ceph-mon source=cloud:focal-xena

keystone
^^^^^^^^

There are three units of the keystone application and its charm supports the
three actions that the paused-single-unit method demands. In addition, the
keystone application is running under HA with the aid of the hacluster
application, which allows for a more controlled upgrade. Application leader
``keystone/0`` is upgraded first:

.. code-block:: none

   juju config keystone action-managed-upgrade=True
   juju config keystone openstack-origin=cloud:focal-xena

   juju run-action --wait keystone-hacluster/0 pause
   juju run-action --wait keystone/0 pause
   juju run-action --wait keystone/0 openstack-upgrade
   juju run-action --wait keystone/0 resume
   juju run-action --wait keystone-hacluster/0 resume

   juju run-action --wait keystone-hacluster/1 pause
   juju run-action --wait keystone/1 pause
   juju run-action --wait keystone/1 openstack-upgrade
   juju run-action --wait keystone/1 resume
   juju run-action --wait keystone-hacluster/1 resume

   juju run-action --wait keystone-hacluster/2 pause
   juju run-action --wait keystone/2 pause
   juju run-action --wait keystone/2 openstack-upgrade
   juju run-action --wait keystone/2 resume
   juju run-action --wait keystone-hacluster/2 resume

ceph-radosgw
^^^^^^^^^^^^

There is only a single unit of the ceph-radosgw application. Use the all-in-one
method:

.. code-block:: none

   juju config ceph-radosgw source=cloud:focal-xena

cinder
^^^^^^

There is only a single unit of the cinder application. Use the all-in-one
method:

.. code-block:: none

   juju config cinder openstack-origin=cloud:focal-xena

glance
^^^^^^

There is only a single unit of the glance application. Use the all-in-one
method:

.. code-block:: none

   juju config glance openstack-origin=cloud:focal-xena

neutron-api
^^^^^^^^^^^

There is only a single unit of the neutron-api application. Use the all-in-one
method:

.. code-block:: none

   juju config neutron-api openstack-origin=cloud:focal-xena

ovn-central
^^^^^^^^^^^

Although there are three units of the ovn-central application, based on the
actions supported by the ovn-central charm, only the all-in-one method is
available:

.. code-block:: none

   juju config ovn-central source=cloud:focal-xena

placement
^^^^^^^^^

There is only a single unit of the placement application. Use the all-in-one
method:

.. code-block:: none

   juju config placement openstack-origin=cloud:focal-xena

nova-cloud-controller
^^^^^^^^^^^^^^^^^^^^^

There is only a single unit of the nova-cloud-controller application. Use the
all-in-one method:

.. code-block:: none

   juju config nova-cloud-controller openstack-origin=cloud:focal-xena

openstack-dashboard
^^^^^^^^^^^^^^^^^^^

There is only a single unit of the openstack-dashboard application. Use the
all-in-one method:

.. code-block:: none

   juju config openstack-dashboard openstack-origin=cloud:focal-xena

nova-compute
^^^^^^^^^^^^

There are three units of the nova-compute application and its charm supports
the three actions that the paused-single-unit method requires. Application
leader ``nova-compute/2`` is upgraded first:

.. code-block:: none

   juju config nova-compute action-managed-upgrade=True
   juju config nova-compute openstack-origin=cloud:focal-xena

   juju run-action --wait nova-compute/2 pause
   juju run-action --wait nova-compute/2 openstack-upgrade
   juju run-action --wait nova-compute/2 resume

   juju run-action --wait nova-compute/1 pause
   juju run-action --wait nova-compute/1 openstack-upgrade
   juju run-action --wait nova-compute/1 resume

   juju run-action --wait nova-compute/0 pause
   juju run-action --wait nova-compute/0 openstack-upgrade
   juju run-action --wait nova-compute/0 resume

ceph-osd
^^^^^^^^

Although there are three units of the ceph-osd application, the all-in-one
method is used because the ceph-osd charm is able to maintain service
availability during the upgrade:

.. code-block:: none

   juju config ceph-osd source=cloud:focal-xena

Re-enable unattended-upgrades
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Re-enable unattended-upgrades on the three cloud nodes:

.. code-block:: none

   juju ssh 0 sudo dpkg-reconfigure -plow unattended-upgrades
   juju ssh 1 sudo dpkg-reconfigure -plow unattended-upgrades
   juju ssh 2 sudo dpkg-reconfigure -plow unattended-upgrades

Answer 'Yes' to resulting the question.

Verify the new deployment
~~~~~~~~~~~~~~~~~~~~~~~~~

Check for errors in :command:`juju status` output and any monitoring service.
Perform a routine battery of tests.

.. LINKS
.. _bundle: https://raw.githubusercontent.com/openstack-charmers/openstack-bundles/b1817add83ba56458aca1aa171ed9b74c211474d/stable/openstack-base/bundle.yaml
