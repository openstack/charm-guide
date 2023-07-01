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
editing a `policy.yaml`_ file on a unit).

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
   will succeed if a keyword was misspelled).

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

Status
------

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

Howto
-----

For practical steps on using policy overrides see the
:doc:`../admin/policy-overrides` page.

.. LINKS
.. _OpenStack service policies: https://docs.openstack.org/security-guide/identity/policies.html
.. _policy.yaml: https://docs.openstack.org/oslo.policy/latest/admin/policy-yaml-file.html
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
