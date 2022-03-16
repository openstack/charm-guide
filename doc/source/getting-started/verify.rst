================
Verify the cloud
================

We'll now verify that the cloud is working by creating a test instance and
connecting to it over SSH.

Create an instance
------------------

Create a Focal amd64 instance called 'focal-1':

.. code-block:: none

   openstack server create \
      --image focal-amd64 --flavor m1.micro --key-name mykey --network int_net \
       focal-1

Assign a floating IP address
----------------------------

Request and assign a floating IP address to the new instance:

.. code-block:: none

   FLOATING_IP=$(openstack floating ip create -f value -c floating_ip_address ext_net)

   openstack server add floating ip focal-1 $FLOATING_IP

Log in to the instance
----------------------

Log in to the new instance:

.. code-block:: none

   ssh -i ~/tutorial/id_mykey ubuntu@$FLOATING_IP

Congratulations, you have a working OpenStack cloud!

Advance to the final step: :doc:`Access the dashboard <dashboard>`.
