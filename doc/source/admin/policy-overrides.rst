================
Policy overrides
================

Overview
--------

`OpenStack service policies`_ define access permissions for resources on a
per-service basis. The policy defaults for a charmed OpenStack service are
either coded in the service itself ("policy-in-code") and/or provided by the
charm via a YAML file.

The preferred approach for modifying a default policy is via charm options as
this leads to consistent policy files. In a Juju-managed OpenStack deployment,
it is not recommended to manually override a service's default policy (i.e.
editing a `policy.json`_ file on a unit).

However, over time the demand has grown for the ability of an operator to tweak
a policy in a way that the charm is currently incapable of, and without
incurring the penalty of waiting for such changes to be implemented in the
charm.

The policy overrides feature provides a mechanism for doing this with the
limitation that it can alter only the permissions of the tenant users of the
system. It does **not** modify the permissions of the service users themselves
(e.g. keystone, glance, nova). The charms maintains its responsibility for this
through the use of policy files within the charms themselves.

Charms
------

Policy overrides are supported on a per-charm basis. This support will be
mentioned in a charm's README along with any charm-specific override
information.

Here is the current list of override-aware charms:

* `cinder`_
* `designate`_
* `glance`_
* `heat`_
* `keystone`_
* `neutron-api`_
* `nova-cloud-controller`_
* `octavia`_
* `openstack-dashboard`_

Overrides for one service may affect the functionality of another service.
Therefore, it may be necessary to provide overrides for multiple services in
order to achieve a consistent set of policies across the cloud. Do not proceed
unless all affected services are represented in the above list.

.. important::

   Always consult the charm documentation prior to using this feature.

Implementation
--------------

Any policy statement valid for a given OpenStack service is placed, one per
line, in a file (an *override file*). This file (or files) is then compressed
into a single file (the *resource file*) and used as an `Application resource`_
named 'policyd-override'. Finally, the override is enabled via a Boolean charm
option.

The enablement phase will cause validation checks to be performed. If
successful, the effective contents of each override file is placed into a
corresponding file under the ``/etc/<service-name>/policy.d/`` directory on the
appropriate unit(s). The service will then use this information to override the
currently active policy.

.. important::

   Validation checks do not cover the policy statements themselves. They only
   ensure that the policy override mechanism is able to consume those
   statements (e.g. failure will result if the statement is not valid YAML but
   will succeed if a keyword was misspelled). See `Requirements`_ below.

A charm may optionally provide a template system where an override file can
include template variables that the charm will substitute with data that the
charm has access to via current charm options, the environment, or relation
data.

The override implementation is per charm. Thus if several services require
overrides a separate resource file will need to be applied to each respective
charm.

.. note::

   A problem arising from overrides is not considered a charm breakage.
   Overrides are orthogonal to the operation of the charm.

Requirements
------------

Requirements for the resource file are presented below.

* It must be properly ZIP formatted. A ``pkunzip`` program must be able to open
  and test the enclosed files.
* Enclosed override files must be properly YAML formatted and have an extension
  of ``.yaml``, or ``.yml``.
* Enclosed override files must **not** contain rule targets/keys that have been
  blacklisted by the charm. These will be documented in the charm's README.
* Enclosed override files must have unique filenames. Any directories in the
  file are "flattened" such that all override files appear as a simple list.
  Each of these filenames also get lower-cased.

Applying overrides
------------------

Policy overrides for a single OpenStack service are applied in the following
way:

#. Insert the policy statements into an override file (or files).

#. Compress the override file(s) to get the resource file:

   .. code-block:: none

      zip <resource-file.zip> <override-file.yaml> [<override-file.yaml> ...]

#. Attach the resource file to the charm corresponding to the service. The
   resource name used is ``policyd-override``:

   .. code-block:: none

      juju attach-resource <charm-name> policyd-override=<resource-file.zip>

#. Enable the override via the ``use-policyd-override`` charm option:

   .. code-block:: none

      juju config <charm-name> use-policyd-override=true

To update (or fix) the overrides simply attach a new resource file. Changes
are applied immediately; there is no need to disable ('false') and re-enable
('true').

.. note::

   The overrides that get applied are always associated with the most recently
   attached resource file.

The last revision time of the resource can be viewed with the :command:`juju
list-resources` command. Sample output is:

.. code-block:: console

   Resource          Revision
   policyd-override  2020-03-12T19:53

Disabling overrides
-------------------

Overrides are disabled by setting option ``use-policyd-override`` back to its
default value of 'false':

.. code-block:: none

   juju config <charm-name> use-policyd-override=false

There is no ability in Juju to remove a resource file.

.. note::

   A charm that supports policy overrides will always have the
   'policyd-override' resource present.

Override status
---------------

The status of enabled overrides for an application is shown in the output for
the :command:`juju status` command. When overrides are successful the text
``PO:`` (Policy Overrides) will be prefixed to the application's status
message. When they are unsuccessful ``PO: (broken)`` will be used.

An unsuccessful override implies that **none** of the override policy
statements have been applied. In this case, the operator should either attach
a fixed resource file or disable the overrides entirely.

Information on broken overrides will appear in the logs for the application in
question. For instance, for nova-cloud-controller:

.. code-block:: none

   juju debug-log --replay --no-tail --include nova-cloud-controller

Examples
--------

This area contains examples of policy override usage.

Showing extended server attributes
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This example involves changing the default policy affecting the
nova-cloud-controller application.

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

To make this happen the default policy affecting the `Nova API`_ will need to
be overridden to include the owner of the instance as well as the admin. The
policy "target" that controls these particular fields is
``os_compute_api:os-extended-server-attributes``.

The final policy statement is placed in a file, say,
``nova-server-attributes.yaml``:

.. code-block:: yaml

   #"os_compute_api:os-extended-server-attributes": "rule:admin_api"
   "os_compute_api:os-extended-server-attributes": "rule:admin_or_owner"

The default statement is left as a comment in order to provide some extra
context.

Compress the file, attach it as a resource to the nova-cloud-controller
application, and enable the override:

.. code-block:: none

   zip nova-server-attributes.zip nova-server-attributes.yaml
   juju attach-resource nova-cloud-controller policyd-override=nova-server-attributes.zip
   juju config nova-cloud-controller use-policyd-override=true

Any non-admin user should now have access to three extra fields when querying
the instances that they own with the :command:`openstack server show` command.

More extended attributes can be displayed through the use of option
``--os-compute-api-version``. For example:

.. code-block:: none

   openstack --os-compute-api-version 2.3 server show 9167b3e9-c653-43fc-858a-2d6f6da36daa

See the upstream documentation on `Show Server Details`_.

.. LINKS
.. _OpenStack service policies: https://docs.openstack.org/security-guide/identity/policies.html
.. _policy.json: https://docs.openstack.org/oslo.policy/latest/admin/policy-json-file.html
.. _Nova API: https://docs.openstack.org/nova/latest/configuration/policy.html
.. _Show Server Details: https://docs.openstack.org/api-ref/compute/?expanded=show-server-details-detail#show-server-details
.. _Application resource: https://juju.is/docs/sdk/resources

.. CHARMS
.. _cinder: https://opendev.org/openstack/charm-cinder/src/branch/master/README.md#user-content-policy-overrides
.. _designate: https://opendev.org/openstack/charm-designate/src/branch/master/src/README.md#user-content-policy-overrides
.. _glance: https://opendev.org/openstack/charm-glance/src/branch/master/README.md#user-content-policy-overrides
.. _heat: https://opendev.org/openstack/charm-heat/src/branch/master/README.md#user-content-policy-overrides
.. _keystone: https://opendev.org/openstack/charm-keystone/src/branch/master/README.md#user-content-policy-overrides
.. _neutron-api: https://opendev.org/openstack/charm-neutron-api/src/branch/master/README.md#user-content-policy-overrides
.. _nova-cloud-controller: https://opendev.org/openstack/charm-nova-cloud-controller/src/branch/master/README.md#user-content-policy-overrides
.. _octavia: https://opendev.org/openstack/charm-octavia/src/branch/master/README.md#user-content-policy-overrides
.. _openstack-dashboard: https://opendev.org/openstack/charm-openstack-dashboard/src/branch/master/README.md#user-content-policy-overrides
