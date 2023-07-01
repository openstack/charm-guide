======================================================
Show extended server attributes using policy overrides
======================================================

This tutorial shows how to show extended server (VM) attributes using policy
overrides. It involves changing the default policy affecting the
nova-cloud-controller application.

.. important::

   It is recommended to read through the howto document on
   :doc:`policy-overrides` prior to acting on the steps provided in this
   document.

Introduction
------------

Ordinarily, when a non-admin user requests details for a cloud instance some
fields are not shown. This is because some information is deemed inappropriate
or too sensitive for the regular user. For instance, this is the (partial)
default output to the :command:`openstack server show` command:

.. code-block:: console

   echo $OS_USERNAME
   User1

   openstack server show 9167b3e9-c653-43fc-858a-2d6f6da36daa

   +-----------------------------+----------------------------------------------------------+
   | Field                       | Value                                                    |
   +-----------------------------+----------------------------------------------------------+
   | OS-DCF:diskConfig           | MANUAL                                                   |
   | OS-EXT-AZ:availability_zone | nova                                                     |
   | OS-EXT-STS:power_state      | Running                                                  |
   | OS-EXT-STS:task_state       | None                                                     |
   | OS-EXT-STS:vm_state         | active                                                   |
   | OS-SRV-USG:launched_at      | 2019-12-11T23:09:47.000000                               |
   | OS-SRV-USG:terminated_at    | None                                                     |

Compare that output to what an admin sees:

.. code-block:: console

   echo $OS_USERNAME
   admin

   openstack server show 9167b3e9-c653-43fc-858a-2d6f6da36daa

   +-------------------------------------+--------------------------------------------------+
   | Field                               | Value                                            |
   +-------------------------------------+--------------------------------------------------+
   | OS-DCF:diskConfig                   | MANUAL                                           |
   | OS-EXT-AZ:availability_zone         | nova                                             |
   | OS-EXT-SRV-ATTR:host                | virt-node-01.maas                                |
   | OS-EXT-SRV-ATTR:hypervisor_hostname | virt-node-01.maas                                |
   | OS-EXT-SRV-ATTR:instance_name       | instance-00000001                                |
   | OS-EXT-STS:power_state              | Running                                          |
   | OS-EXT-STS:task_state               | None                                             |
   | OS-EXT-STS:vm_state                 | active                                           |
   | OS-SRV-USG:launched_at              | 2019-12-11T23:09:47.000000                       |
   | OS-SRV-USG:terminated_at            | None                                             |

The admin user has three extra fields that are categorised as *extended server
attributes*:

.. code-block:: console

   | OS-EXT-SRV-ATTR:host                | virt-node-01.maas                                |
   | OS-EXT-SRV-ATTR:hypervisor_hostname | virt-node-01.maas                                |
   | OS-EXT-SRV-ATTR:instance_name       | instance-00000001                                |

For some environments, such as an internal company cloud, the benefits of
providing this information to users may outweigh any perceived concerns. For
example, users will know immediately whether an announced hypervisor
maintenance procedure will affect their running instances, providing that the
announcement includes the hypervisor name.

Create the override file
------------------------

To make this happen the default policy affecting the `Nova API`_ will need to
be overridden to include the owner of the instance as well as the admin. The
policy "target" that controls these particular fields is
``os_compute_api:os-extended-server-attributes``.

The final policy statement is placed in a file, say,
``nova-server-attributes.yaml``:

.. code-block:: none

   #"os_compute_api:os-extended-server-attributes": "rule:admin_api"
   "os_compute_api:os-extended-server-attributes": "rule:admin_or_owner"

The default statement is left as a comment in order to provide some extra
context.

Compress the override file
--------------------------

Compress the override file to get the resource file, here
``nova-server-attributes.yaml``:

.. code-block:: none

   zip nova-server-attributes.zip nova-server-attributes.yaml

Attach the resource file to the application
-------------------------------------------

Attach the resource file to the nova-cloud-controller application. The resource
name used is always ``policyd-override``:

.. code-block:: none

   juju attach-resource nova-cloud-controller policyd-override=nova-server-attributes.zip

Enable the override
-------------------

Enable the override via the ``use-policyd-override`` charm option:

.. code-block:: none

   juju config nova-cloud-controller use-policyd-override=true

Result
~~~~~~

Any non-admin user should now have access to three extra fields when querying
the instances that they own with the :command:`openstack server show` command.

More extended attributes can be displayed through the use of option
``--os-compute-api-version``. For example:

.. code-block:: none

   openstack --os-compute-api-version 2.3 server show 9167b3e9-c653-43fc-858a-2d6f6da36daa

See the upstream documentation on `Show Server Details`_.

.. LINKS
.. _Nova API: https://docs.openstack.org/nova/latest/configuration/policy.html
.. _Show Server Details: https://docs.openstack.org/api-ref/compute/?expanded=show-server-details-detail#show-server-details
