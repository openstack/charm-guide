:orphan:

========================================
Implement automatic Glance image updates
========================================

Introduction
------------

An OpenStack cloud generally benefits from making available the most recent
cloud images as it minimizes the need to perform software updates on its VMs.
It is also convenient to automate the process of updating these images. This
article will show how to accomplish all this with the
`glance-simplestreams-sync`_ charm.

Requirements
------------

The glance-simplestreams-sync charm places Simplestreams metadata in Object
Storage via the cloud's ``swift`` API endpoint. The cloud will therefore
require the presence of either Swift (`swift-proxy`_ and `swift-storage`_
charms) or the Ceph RADOS Gateway (`ceph-radosgw`_ charm).

Procedure
---------

Deploy the software
~~~~~~~~~~~~~~~~~~~

Deploy the glance-simplestreams-sync application. Here it is containerised on
machine 1:

.. code-block:: none

   juju deploy --to lxd:1 glance-simplestreams-sync
   juju add-relation glance-simplestreams-sync:identity-service keystone:identity-service
   juju add-relation glance-simplestreams-sync:certificates vault:certificates

We are assuming that the cloud is TLS-enabled (hence the Vault relation).

.. note::

   The glance-simplestreams-sync charm sets up its own ``image-stream``
   endpoint. However, it is not utilised in this present scenario. It is
   leveraged when using OpenStack as a backing cloud to Juju.

Configure image downloads
~~~~~~~~~~~~~~~~~~~~~~~~~

The recommended way to configure image downloads is with a YAML file. As an
example, we'll filter on the following:

* Focal and Jammy images
* arm64 and amd64 architectures
* officially released images and daily images
* the latest of each found image (i.e. maximum of one)

To satisfy all the above place the below configuration in, say, file
``~/gsss.yaml``:

.. code-block:: yaml

   glance-simplestreams-sync:
     mirror_list: |
       [{ url: 'http://cloud-images.ubuntu.com/releases/',
          name_prefix: 'ubuntu:released',
          path: 'streams/v1/index.sjson',
          max: 1,
          item_filters: ['arch~(arm64|amd64)', 'ftype~(uefi1.img|uefi.img|disk1.img)', 'release~(focal|jammy)']
        },
        { url: 'http://cloud-images.ubuntu.com/daily/',
          name_prefix: 'ubuntu:daily',
          path: 'streams/v1/index.sjson',
          max: 1,
          item_filters: ['arch~(arm64|amd64)', 'ftype~(uefi1.img|uefi.img|disk1.img)', 'release~(focal|jammy)']
        },
       ]

Now configure the charm by referencing the file:

.. code-block:: none

   juju config --file ~/gsss.yaml glance-simplestreams-sync

.. note::

   If a configuration is not provided the applications's default behaviour is
   to download the latest official amd64 image for each of the last four LTS
   releases.

Enable automatic image updates
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Enable automatic image updates via the ``run`` option. Here we also specify
checks to occur on a weekly basis:

.. code-block:: none

   juju config glance-simplestreams-sync frequency=weekly run=true

Valid frequencies are 'hourly', 'daily', and 'weekly'.

Enabling the ``run`` option will trigger an initial update within 60 seconds.

Perform a manual image sync (optional)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

A manual image sync can optionally be performed with the ``sync-images``
action:

.. code-block:: none

   juju run-action --wait glance-simplestreams-sync/leader sync-images

Sample output:

.. code-block:: console

   unit-glance-simplestreams-sync-0:
     UnitId: glance-simplestreams-sync/0
     id: "32"
     results:
     .
     .
     .
       Stdout: |
         created 2b0dc3a3-bcac-425b-bbd8-aaa6af848ffd: auto-sync/ubuntu-focal-20.04-amd64-server-20221115.1-disk1.img
         created 1a6964bb-f674-438e-a39f-78b5a274bf19: auto-sync/ubuntu-focal-20.04-arm64-server-20221115.1-disk1.img
         created 1ba8780f-90b1-4872-8b30-8b90be158022: auto-sync/ubuntu-jammy-22.04-amd64-server-20221117-disk1.img
         created 8b9075a3-1528-44d1-8462-7152c7a82a02: auto-sync/ubuntu-jammy-22.04-arm64-server-20221117-disk1.img
         created df861e6a-321e-4025-a8b0-952a2acdf733: auto-sync/ubuntu-focal-daily-amd64-server-20221121-disk1.img
         created cbd91693-cde7-45ab-bbca-0c5d761762d2: auto-sync/ubuntu-focal-daily-arm64-server-20221121-disk1.img
         created e160c75a-b063-45b7-b914-a7669b4244a4: auto-sync/ubuntu-jammy-daily-amd64-server-20221120-disk1.img
         created 92c85d89-2695-48d0-b476-486e9576d931: auto-sync/ubuntu-jammy-daily-arm64-server-20221120-disk1.img
     status: completed
     timing:
       completed: 2022-11-22 17:58:01 +0000 UTC
       enqueued: 2022-11-22 17:52:15 +0000 UTC
       started: 2022-11-22 17:52:16 +0000 UTC

This output should reflect the information available via the
:command:`openstack image list` command.

.. LINKS
.. _glance-simplestreams-sync: https://charmhub.io/glance-simplestreams-sync
.. _ceph-radosgw: https://charmhub.io/ceph-radosgw
.. _swift-proxy: https://charmhub.io/swift-proxy
.. _swift-storage: https://charmhub.io/swift-storage
