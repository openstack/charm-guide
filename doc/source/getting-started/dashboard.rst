====================
Access the dashboard
====================

To access the dashboard (Horizon) first obtain its IP address:

.. code-block:: none

   juju status --format=yaml openstack-dashboard | grep public-address | awk '{print $2}' | head -1

In this example, the address is '10.246.114.29'.

The dashboard URL is then: **http://10.246.114.29/horizon**

Now query for the admin password:

.. code-block:: none

   juju run --unit keystone/leader leader-get admin_passwd

   ****************

The final credentials needed to log in are:

| User Name: **admin**
| Password: ****************
| Domain: **admin_domain**

.. tip::

   If this tutorial was performed on a host remote to your browser, you may
   need to use SSH local port forwarding to access Horizon. For example:

   .. code-block:: none

      sudo ssh -i <personal-key> -N -L 8002:10.246.114.29:80 <remote-host>

   In this case, the URL becomes: **http://localhost:8002/horizon**
