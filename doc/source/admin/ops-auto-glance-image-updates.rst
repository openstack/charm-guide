:orphan:

========================================
Implement automatic Glance image updates
========================================

Preamble
--------

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

* Bionic and Focal images
* arm64 and amd64 architectures
* officially released images and daily images
* the latest of each found image (i.e. maximum of one)

To satisfy all the above place the below configuration in, say, file
``~/gss.yaml``:

.. code-block:: yaml

   glance-simplestreams-sync:
     mirror_list: |
       [{ url: 'http://cloud-images.ubuntu.com/releases/',
          name_prefix: 'ubuntu:released',
          path: 'streams/v1/index.sjson',
          max: 1,
          item_filters: ['arch~(arm64|amd64)', 'ftype~(uefi1.img|uefi.img|disk1.img)', 'release~(bionic|focal)']
        },
        { url: 'http://cloud-images.ubuntu.com/daily/',
          name_prefix: 'ubuntu:daily',
          path: 'streams/v1/index.sjson',
          max: 1,
          item_filters: ['arch~(arm64|amd64)', 'ftype~(uefi1.img|uefi.img|disk1.img)', 'release~(bionic|focal)']
        }
       ]

Now configure the charm by referencing the file:

.. code-block:: none

   juju config --file ~/gss.yaml glance-simplestreams-sync

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
         created 12b3415c-8f50-491f-916c-e08ba4da71c5: auto-sync/ubuntu-bionic-18.04-amd64-server-20210720-disk1.img
         created 73ea8a47-1b1f-48cf-b216-c7eba38d96ab: auto-sync/ubuntu-bionic-18.04-arm64-server-20210720-disk1.img
         created 37d7aeff-5ccb-4a4a-9258-f7948df4caa2: auto-sync/ubuntu-focal-20.04-amd64-server-20210720-disk1.img
         created 10acb4a1-ed7d-4a43-b14c-49d646f23b87: auto-sync/ubuntu-focal-20.04-arm64-server-20210720-disk1.img
         created 90d308d3-cf23-49da-a625-c50a55286d94: auto-sync/ubuntu-bionic-daily-amd64-server-20210720-disk1.img
         created aafa3f2b-002b-4b1c-a212-d99d858bf6b7: auto-sync/ubuntu-bionic-daily-arm64-server-20210720-disk1.img
         created 350d4537-cb8d-445b-a62f-6a1ad15ce3b7: auto-sync/ubuntu-focal-daily-amd64-server-20210720-disk1.img
         created 63f75ea0-e55f-499a-92bc-d02f46126834: auto-sync/ubuntu-focal-daily-arm64-server-20210720-disk1.img
     status: completed
     timing:
       completed: 2021-07-23 22:37:28 +0000 UTC
       enqueued: 2021-07-23 22:22:49 +0000 UTC
       started: 2021-07-23 22:22:54 +0000 UTC

This output should reflect the information available via the
:command:`openstack image list` command.

.. LINKS
.. _glance-simplestreams-sync: https://jaas.ai/glance-simplestreams-sync
.. _ceph-radosgw: https://jaas.ai/ceph-radosgw
.. _swift-proxy: https://jaas.ai/swift-proxy
.. _swift-storage: https://jaas.ai/swift-storage
