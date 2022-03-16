============================
Configure the overlay bundle
============================

The deployment will make use of the ``openstack-base`` bundle, which represents
the core of a Charmed OpenStack cloud. A separate overlay bundle is used to
tailor the configuration (possibly overriding the bundle) so that the deploy
will work within a given environment.

Save the below overlay to
``~/tutorial/overlay-openstack-base-focal-mymaas.yaml`` and replace the
variables with the values that were collected in the :doc:`previous step
<settings>`.

.. note::

   If constraints are not being used and/or if network spaces do not apply then
   replace their values ($CONSTRAINTS and/or $EXT_SPACE) with a null value
   (nothing).

.. code-block:: console

   machines:

     '0':
       constraints: $CONSTRAINTS
     '1':
       constraints: $CONSTRAINTS
     '2':
       constraints: $CONSTRAINTS

   variables:

     data-port: &data-port br-ex:$OVN_DATA_PORT
     osd-devices: &osd-devices $OSD_DEVICES
     network-space: &network-space $EXT_SPACE

   applications:

     ovn-chassis:
       options:
         bridge-interface-mappings: *data-port

     ceph-osd:
       options:
         osd-devices: *osd-devices
       bindings:
         "": *network-space

     nova-compute:
       bindings:
         "": *network-space

     vault:
       bindings:
         "": *network-space

     ceph-mon:
       bindings:
         "": *network-space

     ceph-radosgw:
       bindings:
         "": *network-space

     glance:
       bindings:
         "": *network-space

     keystone:
       bindings:
         "": *network-space

     mysql-innodb-cluster:
       bindings:
         "": *network-space

     neutron-api:
       bindings:
         "": *network-space

     nova-cloud-controller:
       bindings:
         "": *network-space

     rabbitmq-server:
       bindings:
         "": *network-space

     placement:
       bindings:
         "": *network-space

     ovn-central:
       bindings:
         "": *network-space

     cinder:
       bindings:
         "": *network-space

     openstack-dashboard:
       bindings:
         "": *network-space

Once you've substituted in your values and saved the file, proceed to the
:doc:`Prepare Juju <juju>` page.
