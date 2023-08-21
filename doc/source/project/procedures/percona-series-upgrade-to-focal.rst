==============================================
percona-cluster charm: series upgrade to focal
==============================================

.. important::

   This page has been identified as being affected by the breaking changes
   introduced between versions 2.9.x and 3.x of the Juju client. Read
   support note :ref:`juju_29_3x_changes` before continuing.

.. note::

   This page describes a procedure that may be required when performing an
   upgrade of an OpenStack cloud. Please read the more general
   :doc:`../../admin/upgrades/overview` before attempting any of the
   instructions given here.

In Ubuntu 20.04 LTS (Focal) the percona-xtradb-cluster-server package will no
longer be available. It has been replaced by mysql-server-8.0 and mysql-router
in Ubuntu main. Therefore, there is no way to series upgrade percona-cluster to
Focal. Instead the databases hosted by percona-cluster will need to be migrated
to mysql-innodb-cluster and mysql-router will need to be deployed as a
subordinate on the applications that use MySQL as a data store.

.. warning::

   Since the DB affects most OpenStack services it is important to have a
   sufficient downtime window. The following procedure is written in an attempt
   to migrate one service at a time (i.e. keystone, glance, cinder, etc).
   However, it may be more practical to migrate all databases at the same time
   during an extended downtime window, as there may be unexpected
   interdependencies between services.

.. note::

   It is possible for percona-cluster to remain on Ubuntu 18.04 LTS while
   the rest of the cloud migrates to Ubuntu 20.04 LTS. In fact, this state
   will be one step of the migration process.

.. caution::

   It is recommended that all machines touched by the migration have their
   system software packages updated prior to the migration. There are reports
   indicating that MySQL authentication problems may arise later if this is not
   done.


Procedure
^^^^^^^^^

* Leave all the percona-cluster machines on Bionic and upgrade the series of
  the remaining machines in the cloud per the
  :doc:`../../admin/upgrades/series-openstack` page.

* Deploy a mysql-innodb-cluster on Focal.

  .. warning::

     If multiple network spaces are used in the deployment, due to the way
     MySQL Innodb clustering promulgates cluster addresses via metadata, the
     following space bindings must be bound to the same network space:

     * Primary application (i.e. keystone)

       * shared-db

     * Application's mysql-router (i.e. keystone-mysql-router)

       * shared-db
       * db-rotuer

     * mysql-innodb-cluster

       * db-router
       * cluster

     Space bindings are configured at application deploy time. Failure to
     ensure this may lead to authentication errors if the client connection
     uses the wrong interface to connect to the cluster. In the example below
     we use space "db-space."

  .. code-block:: none

     juju deploy -n 3 mysql-innodb-cluster --series focal --bind "cluster=db-space db-router=db-space"

  .. note::

     Any existing percona-cluster configuration related to performance tuning
     should be configured on the mysql-innodb-cluster charm also.  Although
     there is not a one-to-one parity of options between the charms, there are
     still several identical ones such as ``max-connections`` and
     ``innodb-buffer-pool-size``.

     .. code-block:: none

        juju config mysql-innodb-cluster max-connections=<value> innodb-buffer-pool-size=<value>

* Deploy (but do not yet relate) an instance of mysql-router for every
  application that requires a data store (i.e. every application that was
  related to percona-cluster).

  .. code-block:: none

     juju deploy mysql-router cinder-mysql-router --bind "shared-db=db-space db-router=db-space"
     juju deploy mysql-router glance-mysql-router --bind "shared-db=db-space db-router=db-space"
     juju deploy mysql-router keystone-mysql-router --bind "shared-db=db-space db-router=db-space"
     ...

* Add relations between the mysql-router instances and the
  mysql-innodb-cluster.

  .. code-block:: none

     juju integrate cinder-mysql-router:db-router mysql-innodb-cluster:db-router
     juju integrate glance-mysql-router:db-router mysql-innodb-cluster:db-router
     juju integrate keystone-mysql-router:db-router mysql-innodb-cluster:db-router
     ...

On a per-application basis:

* Remove the relation between the application charm and the percona-cluster
  charm. You can view existing relations with the :command:`juju status
  percona-cluster --relations` command.

  .. code-block:: none

     juju remove-relation keystone:shared-db percona-cluster:shared-db

* Dump the existing database(s) from percona-cluster.

  .. note::

     In the following, the percona-cluster/0 and mysql-innodb-cluster/0 units
     are used as examples. For percona, any unit of the application may be used,
     though all the steps should use the same unit. For mysql-innodb-cluster,
     the RW unit should be used. The RW unit of the mysql-innodb-cluster can be
     determined from the :command:`juju status mysql-innodb-cluster` command.

  * Allow Percona to dump databases. See `Percona strict mode`_ to understand
    the implications of this setting.

    .. code-block:: none

       juju run --wait percona-cluster/0 set-pxc-strict-mode mode=MASTER

  * Here is a non-exhaustive example that lists databases using the :command:`mysql` client:

    .. code-block:: none

       mysql> SHOW DATABASES;
       +--------------------+
       | Database           |
       +--------------------+
       | information_schema |
       | aodh               |
       | cinder             |
       | designate          |
       | dpm                |
       | glance             |
       | gnocchi            |
       | horizon            |
       | keystone           |
       | mysql              |
       | neutron            |
       | nova               |
       | nova_api           |
       | nova_cell0         |
       | performance_schema |
       | placement          |
       | sys                |
       +--------------------+
       17 rows in set (0.10 sec)

  * Dump the specific application's database(s).

    .. note::

       Depending on downtime restrictions it is possible to dump all OpenStack
       databases at one time: run the ``mysqldump`` action and select them via
       the ``databases`` parameter. For example:
       ``databases=keystone,cinder,glance,nova,nova_api,nova_cell0,horizon``

       Similarly, it is possible to import all the databases into
       mysql-innodb-clulster from that single dump file.

    .. warning::

       Do not (back up and) restore the Percona Cluster version of the 'mysql',
       'performance_schema', 'sys' or any other system specific databases into
       the MySQL Innodb Cluster. Doing so will corrupt the DB and necessitate
       the destruction and re-creation of the mysql-innodb-cluster application.
       For more information see bug `LP #1936210`_.

    .. note::

       The database name may or may not match the application name. For example,
       while keystone has a DB named keystone, openstack-dashboard has a database
       named horizon. Some applications have multiple databases. Notably,
       nova-cloud-controller which has at least: nova,nova_api,nova_cell0 and a
       nova_cellN for each additional cell. See upstream documentation for the
       respective application to determine the database name.

    .. code-block:: none

       # Single DB
       juju run --wait percona-cluster/0 mysqldump databases=keystone

       # Multiple DBs
       juju run --wait percona-cluster/0 mysqldump \
       databases=aodh,cinder,designate,glance,gnochii,horizon,keystone,neutron,nova,nova_api,nova_cell0,placement

  * Return Percona enforcing strict mode. See `Percona strict mode`_ to
    understand the implications of this setting.

    .. code-block:: none

       juju run --wait percona-cluster/0 set-pxc-strict-mode mode=ENFORCING

* Transfer the mysqldump file from the percona-cluster unit to the
  mysql-innodb-cluster RW unit. The RW unit of the mysql-innodb-cluster can be
  determined with :command:`juju status mysql-innodb-cluster`. Bellow we use
  mysql-innodb-cluster/0 as an example.

  .. code-block:: none

     juju scp percona-cluster/0:/var/backups/mysql/mysqldump-keystone-<DATE>.gz .
     juju scp mysqldump-keystone-<DATE>.gz mysql-innodb-cluster/0:/home/ubuntu

* Import the database(s) into mysql-innodb-cluster.

  .. code-block:: none

     juju run --wait mysql-innodb-cluster/0 restore-mysqldump dump-file=/home/ubuntu/mysqldump-keystone-<DATE>.gz

* Relate an instance of mysql-router for every application that requires a data
  store (i.e. every application that needed percona-cluster):

  .. code-block:: none

     juju integrate keystone:shared-db keystone-mysql-router:shared-db

* Repeat for remaining applications.

An overview of this process can be seen in the OpenStack charmer's team CI
`Zaza migration code`_.

Post-migration
^^^^^^^^^^^^^^

As noted above, it is possible to run the cloud with percona-cluster remaining
on Bionic indefinitely. Once all databases have been migrated to
mysql-innodb-cluster, all the databases have been backed up, and the cloud has
been verified to be in good working order the percona-cluster application (and
its probable hacluster subordinates) may be removed.

.. code-block:: none

   juju remove-application percona-cluster-hacluster
   juju remove-application percona-cluster

.. LINKS
.. _Zaza migration code: https://github.com/openstack-charmers/zaza-openstack-tests/blob/master/zaza/openstack/charm_tests/mysql/tests.py#L556
.. _Percona strict mode: https://www.percona.com/doc/percona-xtradb-cluster/LATEST/features/pxc-strict-mode.html
.. _`LP #1936210`: https://bugs.launchpad.net/charm-deployment-guide/+bug/1936210
