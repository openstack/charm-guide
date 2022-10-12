============================
Configure the overlay bundle
============================

The deployment will make use of the ``openstack-base`` bundle, which represents
the core of a Charmed OpenStack cloud. A separate overlay bundle is used to
tailor the configuration (possibly overriding settings in the bundle) so that
the deploy will work within a given environment.

.. note::

   Although it is possible to edit the bundle file itself, it is best practice
   to keep it pristine and to use an overlay instead.

Download overlay file :download:`overlay-focal-yoga-mymaas.yaml` and save it in
the ``~/tutorial`` directory.

Replace the variables in the file with the some of the values that were
collected in the :doc:`previous step <settings>`.

Once you've edited and saved the file, proceed to the :doc:`juju` page.

Adjustments
-----------

The overlay file will require further adjustments as per the below scenarios.

Constraints
~~~~~~~~~~~

Replace ``$CONSTRAINTS`` with a null value (nothing) if constraints are not
being used.

Network spaces
~~~~~~~~~~~~~~

Each principal charm will need to be informed of a network space, via the
``bindings`` charm parameter, if a network space is configured within MAAS for
the cloud's subnet.

OVN Chassis data port and hardware addresses
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Comment out the ``data-port`` variable if hardware addresses are being used to
designate the OVN Chassis interfaces:

.. code-block:: none

   #data-port: &data-port br-ex:$OVN_DATA_PORT

Charm option ``bridge-interface-mappings`` should then be configured in this
way:

.. code-block:: yaml

   bridge-interface-mappings: >-
     br-ex:52:54:00:03:01:01
     br-ex:52:54:00:03:01:02
     br-ex:52:54:00:03:01:03

Use your own MAC addresses but preserve the ``br-ex:`` portion.

Example overlay
~~~~~~~~~~~~~~~

Below is an example overlay bundle that has been adjusted according to the
local environment.

.. code-block:: yaml

   machines:

     '0':
       constraints: tags=ovs-bridge
     '1':
       constraints: tags=ovs-bridge
     '2':
       constraints: tags=ovs-bridge

   variables:

     data-port: &data-port br-ex:enp1s0
     osd-devices: &osd-devices /dev/sda /dev/sdb /dev/sdc /dev/sdd

   applications:

     ovn-chassis:
       options:
         bridge-interface-mappings: *data-port

     ceph-osd:
       options:
         osd-devices: *osd-devices
       bindings:
         "": public-space

     ceph-mon:
       bindings:
         "": public-space

     .
     .
     .
