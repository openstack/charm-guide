:orphan:

============================
OpenStack charms and bundles
============================

This page includes charm and bundle information that is specific to the
OpenStack Charms project.

Documentation
-------------

The documentation of every OpenStack charm and bundle is provided in its
repository README file. For instance, see the `keystone charm README`_.

Support
-------

The OpenStack charms support both multiple OpenStack versions and multiple
Ubuntu releases. The currently supported combinations of OpenStack and Ubuntu
are listed on the `Ubuntu Cloud Archive`_ wiki page.

The full list of supported charms is given on the :doc:`Charms
<../project/openstack-charms>` page in this guide.

Charm Store namespaces
----------------------

The OpenStack charms and bundles are available through the online `Charm
Store`_. The promulgated (stable) namespace is `openstack-charmers`_ and the
development namespace is `openstack-charmers-next`_. Follow those links to view
the charms and bundles available in the Store.

There are therefore two Charm Store namespaces to deploy from. Note that the
promulgated namespace is the default namespace:

* stable charm: ``cs:<charm or bundle>``
* development charm: ``cs:~openstack-charmers-next/<charm or bundle>``

.. note::

   The ``cs:`` portion of a charm or bundle URL means "charm store".

Deploying a charm
-----------------

Here are the deployment commands for both the stable and development keystone
charm on Focal (the ``cs:`` can be omitted for the promulgated namespace):

.. code-block:: none

   juju deploy --series focal keystone
   juju deploy --series focal cs:~openstack-charmers-next/keystone

.. tip::

   The series specification is not needed if the Juju model has a default
   series configured.

Deploying a bundle
------------------

Charm bundles can also be deployed directly from the Charm Store:

.. code-block:: none

   juju deploy ceph-base

.. note::

   Only the (stable) `promulgated bundles`_ should be deployed from the Store
   as the development bundles in the Store are not always updated.

Working with bundles
~~~~~~~~~~~~~~~~~~~~

Generally, due to their inherent complexity, OpenStack bundles are downloaded
for viewing prior to deployment. They are then either edited locally or used in
conjunction with an overlay bundle. The easiest way to access the OpenStack
bundles locally is to clone the `openstack-bundles`_ repository:

.. code-block:: none

   git clone https://github.com/openstack-charmers/openstack-bundles

The stable bundles are found under the ``stable`` directory. Overlay bundles
are also available (e.g. ``stable/overlays``).

A bundle can use the same above Store namespaces for the charms contained
within it. The below bundle (``bundle.yaml``) deploys, on Focal, Keystone and a
cloud database and then adds the necessary relations. Only the keystone charm
is using a development version.

.. code-block:: yaml

   series: focal

   machines:

     '0':
     '1':
     '2':
     '3':

   applications:

     mysql-innodb-cluster:
       charm: cs:mysql-innodb-cluster
       num_units: 3
       to:
       - lxd:0
       - lxd:1
       - lxd:2

     keystone-mysql-router:
       charm: cs:mysql-router

     keystone:
       charm: cs:~openstack-charmers-next/keystone
       num_units: 1
       to:
       - lxd:3

   relations:

   - - keystone-mysql-router:db-router
     - mysql-innodb-cluster:db-router
   - - keystone-mysql-router:shared-db
     - keystone:shared-db

.. important::

   In a bundle, a charm's URL must include the ``cs:`` portion.

Here is a simple bundle deployment command:

.. code-block:: none

   juju deploy ./bundle.yaml

.. LINKS
.. _Charm Store: https://jaas.ai
.. _Ubuntu Cloud Archive: https://wiki.ubuntu.com/OpenStack/CloudArchive
.. _Juju Charm Store: https://jujucharms.com/q/?text=openstack
.. _keystone charm README: https://opendev.org/openstack/charm-keystone/src/branch/master/README.md
.. _openstack-charmers: https://jaas.ai/u/openstack-charmers
.. _openstack-charmers-next: https://jaas.ai/u/openstack-charmers-next
.. _promulgated bundles: https://jaas.ai/u/openstack-charmers#bundles
.. _openstack-bundles: https://github.com/openstack-charmers/openstack-bundles
