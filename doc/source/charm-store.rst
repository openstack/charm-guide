:orphan:

.. _charm_store:

==============================
Deploying from the Charm Store
==============================

All of the OpenStack charms, and related, charms are stored in the `Juju Charm
Store`_ which is a repository from which to deploy charms.

.. _Juju Charm Store: https://jujucharms.com/q/?text=openstack

Targets in the Charm Store
==========================

You can deploy from either of the following targets in the charm store.

* For stable charms: ``cs:<charm name>``
* For development charms: ``cs:~openstack-charmers-next/<charm name>``

Note that OpenStack charms support multiple OpenStack versions *and* multiple
Ubuntu releases.  The currently supported versions and combinations can be
found at the `Ubuntu Cloud Archive`_ page and `Ubuntu release end of life`_
pages.

.. _Ubuntu Cloud Archive: https://wiki.ubuntu.com/OpenStack/CloudArchive
.. _Ubuntu release end of life: https://www.ubuntu.com/info/release-end-of-life

e.g. ``cs:xenial/keystone`` is the stable charm in the 16.04 Xenial release of
ubuntu, whereas ``cs:~openstack-charmers-next/keystone`` is the current
development version (against Xenial).

Note that OpenStack development team develops against the current released LTS
version of Ubuntu, and the current development release of Ubuntu.

Example Bundle
~~~~~~~~~~~~~~

.. code-block:: yaml

    # vim: set ts=2 et:
    # deployer bundle for development ('next') charms
    openstack-services:
      series: trusty
      services:
        mysql:
          branch: cs:~openstack-charmers-next/percona-cluster
          constraints: mem=1G
          options:
            dataset-size: 50%
        rabbitmq-server:
          branch: cs:~openstack-charmers-next/rabbitmq-server
          constraints: mem=1G
        keystone:
          branch: cs:~openstack-charmers-next/keystone
          constraints: mem=1G
          options:
            admin-password: openstack
            admin-token: ubuntutesting
            openstack-origin: cloud:xenial-mitaka
        barbican:
          charm: barbican
          options:
            openstack-origin: cloud:trusty-kilo
      relations:
        - [ keystone, mysql ]
        - [ barbican, mysql ]
        - [ barbican, rabbitmq-server ]
        - [ barbican, keystone ]

In this example the bundle would be used to deploy the 'next' charms (i.e.
those in development) with a local barbican charm.
