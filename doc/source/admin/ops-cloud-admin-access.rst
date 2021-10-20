:orphan:

==============================
Set up admin access to a cloud
==============================

Preamble
--------

In order to configure a newly deployed OpenStack cloud for production use one
must first gain native administrative control of it. Although this refers to
OpenStack-level admin user access, this article will show how to obtain it via
queries made with the Juju client.

.. note::

   As an alternative to the instructions presented in this article, if the
   Horizon dashboard is available, access can be obtained by downloading a
   credentials file.

Procedure
---------

Install the client software
~~~~~~~~~~~~~~~~~~~~~~~~~~~

The OpenStack clients will be needed in order to manage the cloud from the
command line. Install them on the same machine that hosts the Juju client. This
example uses the snap install method:

.. code-block:: none

   sudo snap install openstackclients --classic

Set cloud-specific authentication variables
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In terms of authentication, three cloud-specific pieces of information are
needed:

* the Keystone administrator password
* the Keystone service endpoint
* the root CA certificate (if the cloud is TLS-enabled)

Keystone administrator password
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Set environmental variable ``OS_PASSWORD`` to the Keystone administrator
password:

.. code-block:: none

   export OS_PASSWORD=$(juju run --unit keystone/leader 'leader-get admin_passwd')

Keystone service endpoint
^^^^^^^^^^^^^^^^^^^^^^^^^

Determine the IP address of the keystone unit and set environmental variable
``OS_AUTH_URL`` to the Keystone service endpoint:

.. code-block:: none

   IP_ADDRESS=$(juju run --unit keystone/leader -- 'network-get --bind-address public')
   export OS_AUTH_URL=https://${IP_ADDRESS}:5000/v3

.. important::

   If the Keystone endpoint is not using TLS you will need to modify the URL to
   use HTTP.

Root CA certificate
^^^^^^^^^^^^^^^^^^^

Place the CA certificate in a file that your OpenStack client software can
access and set environmental variable ``OS_CACERT`` to that file's path. A
commonly used path that works for the ``openstackclients`` snap, for user
'ubuntu', is ``/home/ubuntu/snap/openstackclients/common/root-ca.crt``:

.. code-block:: none

   export OS_CACERT=/home/ubuntu/snap/openstackclients/common/root-ca.crt
   juju run --unit vault/leader 'leader-get root-ca' > $OS_CACERT

Set other authentication variables
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Charmed OpenStack uses standard values for other authentication variables:

.. code-block:: none

   export OS_USERNAME=admin
   export OS_PROJECT_NAME=admin
   export OS_PROJECT_DOMAIN_NAME=admin_domain
   export OS_USER_DOMAIN_NAME=admin_domain

Verify administrative control
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The admin user environment should now be complete.

First inspect all the variables:

.. code-block:: none

   env | grep OS_

A good initial verification test is to query the cloud's endpoints (Keystone
service catalog):

.. code-block:: none

   openstack endpoint list

A second recommended verification to make is a login to the Horizon dashboard
(if present), where the following should be used:

.. code-block:: console

   OS_USERNAME (User Name)
   OS_PASSWORD (Password)
   OS_PROJECT_DOMAIN_NAME (Domain)

You should now have the permissions to configure and manage the cloud.

Consider a helper script
~~~~~~~~~~~~~~~~~~~~~~~~

Variables can be conveniently set through the use of a shell script that you
can write yourself. However, the OpenStack Charms project maintains such files
(one script calls another) and they can be found in the `openstack-bundles`_
repository.

Simply download the repository and source the ``openrc`` file:

.. code-block:: none

   git clone https://github.com/openstack-charmers/openstack-bundles ~/openstack-bundles
   source ~/openstack-bundles/stable/openstack-base/openrc

This sets a suite of variables. Here is an example:

.. code-block:: console

   OS_REGION_NAME=RegionOne
   OS_AUTH_VERSION=3
   OS_CACERT=/home/ubuntu/snap/openstackclients/common/root-ca.crt
   OS_AUTH_URL=https://10.0.0.162:5000/v3
   OS_PROJECT_DOMAIN_NAME=admin_domain
   OS_AUTH_PROTOCOL=https
   OS_USERNAME=admin
   OS_AUTH_TYPE=password
   OS_USER_DOMAIN_NAME=admin_domain
   OS_PROJECT_NAME=admin
   OS_PASSWORD=aegoaquoo1veZae6
   OS_IDENTITY_API_VERSION=3

Some of the above variables were not covered in the manual method but can be
required in certain situations. For instance, Swift needs ``OS_AUTH_VERSION``,
Gnocchi looks for ``OS_AUTH_TYPE``, and when backing Juju with OpenStack one
needs to know the values of multiple variables (see cloud operation :doc:`Use
OpenStack as a backing cloud for Juju <ops-use-openstack-to-back-juju>`).

.. note::

   The helper files will set the Keystone endpoint variable ``OS_AUTH_URL`` to
   use HTTPS if Vault is detected as containing a root CA certificate. This
   will always be the case due to the OVN requirement for TLS via Vault. If
   Keystone is not TLS-enabled (for some reason) you will need to manually
   reset the above variable to use HTTP.

.. LINKS
.. _openstack-bundles: https://github.com/openstack-charmers/openstack-bundles
