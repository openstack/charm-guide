:orphan:

.. _cinder_volume_replication_custom_overlay:

========================================
Cinder volume replication custom overlay
========================================

The below bundle overlay is used in the instructions given on the :doc:`Cinder
volume replication <cinder-replication>` page.

.. code-block:: yaml

   series: focal

   # Change these variables according to the local environment, 'osd-devices'
   # and 'data-port' in particular.
   variables:
     openstack-origin: &openstack-origin cloud:focal-victoria
     osd-devices: &osd-devices /dev/sdb /dev/vdb
     expected-osd-count: &expected-osd-count 3
     expected-mon-count: &expected-mon-count 3
     data-port: &data-port br-ex:ens7

   relations:
   - - cinder-ceph-a:storage-backend
     - cinder:storage-backend
   - - cinder-ceph-b:storage-backend
     - cinder:storage-backend

   - - site-a-ceph-osd:mon
     - site-a-ceph-mon:osd
   - - site-b-ceph-osd:mon
     - site-b-ceph-mon:osd

   - - site-a-ceph-mon:client
     - nova-compute:ceph
   - - site-b-ceph-mon:client
     - nova-compute:ceph

   - - site-a-ceph-mon:client
     - cinder-ceph-a:ceph
   - - site-b-ceph-mon:client
     - cinder-ceph-b:ceph

   - - nova-compute:ceph-access
     - cinder-ceph-a:ceph-access
   - - nova-compute:ceph-access
     - cinder-ceph-b:ceph-access

   - - site-a-ceph-mon:client
     - glance:ceph

   - - site-a-ceph-mon:rbd-mirror
     - site-a-ceph-rbd-mirror:ceph-local
   - - site-b-ceph-mon:rbd-mirror
     - site-b-ceph-rbd-mirror:ceph-local

   - - site-a-ceph-mon
     - site-b-ceph-rbd-mirror:ceph-remote
   - - site-b-ceph-mon
     - site-a-ceph-rbd-mirror:ceph-remote

   - - site-a-ceph-mon:client
     - cinder-ceph-b:ceph-replication-device
   - - site-b-ceph-mon:client
     - cinder-ceph-a:ceph-replication-device

   applications:

     # Prevent some applications in the main bundle from being deployed.
     ceph-radosgw:
     ceph-osd:
     ceph-mon:
     cinder-ceph:

     # Deploy ceph-osd applications with the appropriate names.
     site-a-ceph-osd:
       charm: cs:ceph-osd
       num_units: 3
       options:
         osd-devices: *osd-devices
         source: *openstack-origin

     site-b-ceph-osd:
       charm: cs:ceph-osd
       num_units: 3
       options:
         osd-devices: *osd-devices
         source: *openstack-origin

     # Deploy ceph-mon applications with the appropriate names.
     site-a-ceph-mon:
       charm: cs:ceph-mon
       num_units: 3
       options:
         expected-osd-count: *expected-osd-count
         monitor-count: *expected-mon-count
         source: *openstack-origin

     site-b-ceph-mon:
       charm: cs:ceph-mon
       num_units: 3
       options:
         expected-osd-count: *expected-osd-count
         monitor-count: *expected-mon-count
         source: *openstack-origin

     # Deploy cinder-ceph applications with the appropriate names.
     cinder-ceph-a:
       charm: cs:cinder-ceph
       num_units: 0
       options:
         rbd-mirroring-mode: image

     cinder-ceph-b:
       charm: cs:cinder-ceph
       num_units: 0
       options:
         rbd-mirroring-mode: image

     # Deploy ceph-rbd-mirror applications with the appropriate names.
     site-a-ceph-rbd-mirror:
       charm: cs:ceph-rbd-mirror
       num_units: 1
       options:
         source: *openstack-origin

     site-b-ceph-rbd-mirror:
       charm: cs:ceph-rbd-mirror
       num_units: 1
       options:
         source: *openstack-origin

     # Configure for the local environment.
     ovn-chassis:
       options:
         bridge-interface-mappings: *data-port
