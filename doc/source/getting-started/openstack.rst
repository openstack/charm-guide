===================
Configure OpenStack
===================

Now that OpenStack is deployed it must now be configured for it to become
functional. Use the values collected on the :doc:`Collect local settings
<settings>` page.

Install the OpenStack clients
-----------------------------

You'll need the OpenStack clients in order to manage the cloud from the
command line. Install them now:

.. code-block:: none

   sudo snap install openstackclients

Access the cloud
----------------

This :download:`openrc <openrc>` file will assist in setting up admin access to
the cloud. Download it under ``~/tutorial``. Then source it and test cloud
access by querying Keystone:

.. code-block:: none

   source ~/tutorial/openrc
   openstack service list

You should get a listing of registered cloud services:

.. code-block:: console

   +----------------------------------+-----------+--------------+
   | ID                               | Name      | Type         |
   +----------------------------------+-----------+--------------+
   | 1510cd32376e4b2783970c292255fee2 | cinderv3  | volumev3     |
   | 1e3f5eb0e1e24d82a683d421adbba85c | cinderv2  | volumev2     |
   | 27fadff76abe4f829a25081aa8bbd98b | placement | placement    |
   | 685053e8c6f04ccc992ac1809437d4e5 | nova      | compute      |
   | 8e65d64be77240539e4d44409aa3bbca | s3        | s3           |
   | 94e467ff95124e9c8b4c608077e61376 | glance    | image        |
   | aeba7526d4064b2f97e9f5c72e0688c1 | keystone  | identity     |
   | b79d5dddc89847419c131deaf333daf1 | neutron   | network      |
   | f1d4699a8bbd40b793a151ecb3ca8de6 | swift     | object-store |
   +----------------------------------+-----------+--------------+

Import an image
---------------

Import a boot image into Glance in order to create instances.

First download a Focal amd64 image:

.. code-block:: none

   curl http://cloud-images.ubuntu.com/focal/current/focal-server-cloudimg-amd64.img \
      --output ~/tutorial/focal-amd64.img

Now import it (calling it 'focal-amd64'):

.. code-block:: none

   openstack image create \
      --public --container-format bare --disk-format qcow2 \
      --file ~/tutorial/focal-amd64.img \
      focal-amd64

Configure networking
--------------------

Create the external network and external subnet:

.. code-block:: none

   openstack network create \
      --external --share --default \
      --provider-network-type flat --provider-physical-network physnet1 \
      ext_net

   openstack subnet create \
      --allocation-pool start=$EXT_POOL_START,end=$EXT_POOL_END \
      --subnet-range $EXT_SUBNET --no-dhcp --gateway $EXT_GW --network ext_net \
      ext_subnet

Create the internal network and internal subnet:

.. code-block:: none

   openstack network create --internal int_net

   openstack subnet create \
      --allocation-pool start=192.168.0.10,end=192.168.0.99 \
      --subnet-range 192.168.0.0/24 --dns-nameserver $EXT_DNS --network int_net \
      int_subnet

Create the router and configure it:

.. code-block:: none

   openstack router create router1

   openstack router add subnet router1 int_subnet

   openstack router set router1 --external-gateway ext_net

Create a flavor
---------------

Create at least one flavor to define a hardware profile for new instances.
Here, to save resources, we create a minimal one called 'm1.micro':

.. code-block:: none

   openstack flavor create \
      --ram 320 --disk 5 --vcpus 1 \
      m1.micro

If you define a larger flavor make sure that your MAAS nodes can accommodate
it.

Import an SSH keypair
---------------------

An SSH keypair needs to be imported into the cloud in order to access your
instances.

Generate one first if you do not yet have one. This command creates a
passphraseless keypair (remove the ``-N`` option to avoid that):

.. code-block:: none

   ssh-keygen -q -N '' -f ~/tutorial/id_mykey

To import a keypair:

.. code-block:: none

   openstack keypair create --public-key ~/tutorial/id_mykey.pub mykey

Configure security groups
-------------------------

To access instances over SSH create a rule for each existing security group:

.. code-block:: none

   for i in $(openstack security group list | awk '/default/{ print $2 }'); do
      openstack security group rule create $i --protocol tcp --remote-ip 0.0.0.0/0 --dst-port 22;
   done

Proceed to the :doc:`Verify the cloud <verify>` page.
