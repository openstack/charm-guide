========================
Upgrade: Queens to Rocky
========================

This page contains notes specific to the Queens to Rocky upgrade path. See the
main :doc:`../../admin/upgrades/openstack` page for full coverage.

Keystone and Fernet tokens
--------------------------

Starting with OpenStack Rocky only the Fernet format for authentication tokens
is supported. Therefore, prior to upgrading Keystone to Rocky a transition must
be made from the legacy format (of UUID) to Fernet.

Fernet support is available upstream (and in the keystone charm) starting with
Ocata so the transition can be made on either Ocata, Pike, or Queens.

A keystone charm upgrade will not alter the token format. The charm's
``token-provider`` option must be used to make the transition:

.. code-block:: none

   juju config keystone token-provider=fernet

This change may result in a minor control plane outage but any running
instances will remain unaffected.

The ``token-provider`` option has no effect starting with Rocky, where the
charm defaults to Fernet and where upstream removes support for UUID. See
`Keystone Fernet Token Implementation`_ for more information.

.. LINKS
.. _Keystone Fernet Token Implementation: https://specs.openstack.org/openstack/charm-specs/specs/rocky/implemented/keystone-fernet-tokens.html
