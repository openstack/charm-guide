====================
Access the dashboard
====================

To access the dashboard (Horizon) first obtain its IP address:

.. code-block:: none

   juju status --format=yaml openstack-dashboard | grep public-address | awk '{print $2}' | head -1

In this example, the address is '10.246.114.75'.

The password can be queried with:

.. code-block:: none

   juju run --unit keystone/leader leader-get admin_passwd

The dashboard URL then becomes:

**http://10.246.114.75/horizon**

The final credentials needed to log in are:

| User Name: **admin**
| Password: ****************
| Domain: **admin_domain**
