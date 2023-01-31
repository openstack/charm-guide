================
Verify the cloud
================

We'll now verify that the cloud is working by creating a test VM and connecting
to it over SSH.

Create a VM
-----------

Create a Jammy amd64 VM called 'jammy-1':

.. code-block:: none

   openstack server create \
      --image jammy-amd64 --flavor m1.micro --key-name mykey --network int_net \
       jammy-1

Assign a floating IP address
----------------------------

Request and assign a floating IP address to the new VM:

.. code-block:: none

   FLOATING_IP=$(openstack floating ip create -f value -c floating_ip_address ext_net)

   openstack server add floating ip jammy-1 $FLOATING_IP

Log in to the VM
----------------

Log in to the new VM:

.. code-block:: none

   ssh -i ~/tutorial/id_mykey ubuntu@$FLOATING_IP

Congratulations, you have a working OpenStack cloud!

Advance to the final step: :doc:`dashboard`.
