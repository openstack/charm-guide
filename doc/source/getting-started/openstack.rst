===================
Configure OpenStack
===================

Now that OpenStack is deployed, in order for it to become functional you must
configure it. Use the values collected on the :doc:`settings` page.

Install the OpenStack clients
-----------------------------

You'll need the OpenStack clients in order to manage the cloud from the
command line. Install them now:

.. code-block:: none

   sudo snap install openstackclients

Access the cloud
----------------

Download cloud init file :download:`openrc <openrc>` and save it in the
``~/tutorial`` directory. It will assist you in setting up admin access to the
cloud.

Now source the file and test cloud access by querying the Keystone service
catalogue:

.. code-block:: none

   source ~/tutorial/openrc
   openstack service list

You should get a listing of registered cloud services:

.. code-block:: console

   +----------------------------------+-----------+--------------+
   | ID                               | Name      | Type         |
   +----------------------------------+-----------+--------------+
   | 2bc94f2e4adc4596a23843311b929748 | swift     | object-store |
   | 2caa7c057158428ca89090314293081a | glance    | image        |
   | 5b4a922d629b4704ad0d634d6ec68c6c | placement | placement    |
   | 6b1f2d914f7548f09718e630773616d3 | s3        | s3           |
   | 99aa18a7eab94560ba11b445b32818f0 | neutron   | network      |
   | b94900f898d54f9c8e77f3f65b64ba66 | nova      | compute      |
   | ece643f6b65a4b57a98cb689cf54139b | keystone  | identity     |
   | f76d49932bcd4801aaca9ccb47e6f5bb | cinderv3  | volumev3     |
   +----------------------------------+-----------+--------------+

Import an image
---------------

You will need a boot image in order to create VMs.

First download a Focal amd64 image:

.. code-block:: none

   wget http://cloud-images.ubuntu.com/focal/current/focal-server-cloudimg-amd64.img \
      -O ~/tutorial/focal-amd64.img

Then import it into Glance. Here we've called it 'focal-amd64':

.. code-block:: none

   openstack image create \
      --public --container-format bare --disk-format qcow2 \
      --file ~/tutorial/focal-amd64.img \
      focal-amd64

Configure networking
--------------------

We'll create internal networking so that OpenStack can assign internal IP
addresses to the VMs it creates. We'll also create external networking that
will allow access to those VMs from outside the cloud. A router is used to
connect the two together.

Create the external network and external subnet. We've called them 'ext_net'
and 'ext_subnet' respectively:

.. code-block:: none

   openstack network create \
      --external --share --default \
      --provider-network-type flat --provider-physical-network physnet1 \
      ext_net

   openstack subnet create \
      --allocation-pool start=$EXT_POOL_START,end=$EXT_POOL_END \
      --subnet-range $EXT_SUBNET --no-dhcp --gateway $EXT_GW --network ext_net \
      ext_subnet

Create the internal network and internal subnet. We've called them 'int_net'
and 'int_subnet' respectively:

.. code-block:: none

   openstack network create --internal int_net

   openstack subnet create \
      --allocation-pool start=192.168.0.10,end=192.168.0.99 \
      --subnet-range 192.168.0.0/24 --dns-nameserver $EXT_DNS --network int_net \
      int_subnet

Create the router. Here we've called it 'router1':

.. code-block:: none

   openstack router create router1

Then connect the router to the internal subnet and set the external network as
its default gateway.

.. code-block:: none

   openstack router add subnet router1 int_subnet

   openstack router set router1 --external-gateway ext_net

Create a flavor
---------------

Create at least one flavor to define a hardware profile for new VMs. Here, to
save resources, we create a minimal one called 'm1.micro':

.. code-block:: none

   openstack flavor create \
      --ram 320 --disk 5 --vcpus 1 \
      m1.micro

If you define a larger flavor make sure that your MAAS nodes can accommodate
it.

Import an SSH keypair
---------------------

An SSH keypair needs to be imported into the cloud in order to access your
VMs.

Generate one first if you do not yet have one. This command creates a
passphraseless keypair (remove the ``-N`` option to avoid that):

.. code-block:: none

   ssh-keygen -q -N '' -f ~/tutorial/id_mykey

To import a keypair:

.. code-block:: none

   openstack keypair create --public-key ~/tutorial/id_mykey.pub mykey

Configure security groups
-------------------------

To access VMs over SSH, create a rule for each existing security group:

.. code-block:: none

   for i in $(openstack security group list | awk '/default/{ print $2 }'); do
      openstack security group rule create $i --protocol tcp --remote-ip 0.0.0.0/0 --dst-port 22;
   done

Proceed to the :doc:`verify` page.
