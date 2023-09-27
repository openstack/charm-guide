:orphan:

===============================
Configure TLS for the Vault API
===============================

.. important::

   This page has been identified as being affected by the breaking changes
   introduced between versions 2.9.x and 3.x of the Juju client. Read
   support note :ref:`juju_29_3x_changes` before continuing.

Introduction
------------

Configuring the Vault API with TLS assures the identity of the Vault service
and encrypts all the information Vault sends over the network. For instance,
unsealing keys will not be sent in cleartext. Note that the issuing of its own
certificates to the various cloud API services (e.g Cinder, Glance, etc.) is
done over relations.

This procedure can also be used to re-configure an already-encrypted Vault API
endpoint.

.. warning::

   This procedure will cause Vault to become sealed. Please ensure that the
   requisite number of unseal keys are available before continuing.

.. caution::

   Although this procedure will cause Vault to become inaccessible, a cloud
   service outage will not occur unless Vault is solicited. Examples of this
   include:

   * new certificates being re-issued via the ``reissue-certificates`` vault
     charm action

   * the rebooting of a Compute node (resulting in the need to decrypt its VMs
     disk locations)

TLS material
~~~~~~~~~~~~

It is assumed that the necessary TLS material exists and that it is stored in
the below locations (the current user is 'ubuntu' in this example):

* ``/home/ubuntu/tls/server.crt`` (server certificate)
* ``/home/ubuntu/tls/server.key`` (server private key)
* ``/home/ubuntu/tls/ca.crt`` (CA certificate)

.. important::

   The CA certificate must be in the current user's home directory for it to be
   available to the vault snap at the subsequent unseal step.

In this example, the Common Name (CN) provided to the server certificate is
'vault.example.com'. Ensure that the hostname is resolvable, from the local
system, to the IP address as reported by Juju (``juju status vault``).

For Vault in HA, there would be a unique server certificate for each unit.

Add TLS material to Vault
~~~~~~~~~~~~~~~~~~~~~~~~~

Add the base64-encoded TLS material to Vault via charm configuration options:

.. code-block:: none

   juju config vault ssl-ca="$(base64 /home/ubuntu/tls/ca.crt)"
   juju config vault ssl-cert="$(base64 /home/ubuntu/tls/server.crt)"
   juju config vault ssl-key="$(base64 /home/ubuntu/tls/server.key)"

Confirm the change by inspecting a file on the unit(s). Once everything has
settled, the server certificate can be found in
``/var/snap/vault/common/vault.crt``:

.. code-block:: none

   juju exec -a vault "sudo cat /var/snap/vault/common/vault.crt"

Restart the Vault service
~~~~~~~~~~~~~~~~~~~~~~~~~

Restart **each** vault unit for its new certificate to be recognised.

.. important::

   Restarting Vault will cause it to become sealed.

For a single unit (``vault/0``):

.. code-block:: none

   juju run vault/0 restart

The output to :command:`juju status vault` should show that Vault is sealed:

.. code-block:: console

   Unit      Workload  Agent  Machine  Public address  Ports     Message
   vault/0*  blocked   idle   3/lxd/3  10.0.0.204      8200/tcp  Unit is sealed

Vault is now configured with the new certificate.

Unseal Vault
~~~~~~~~~~~~

Unseal **each** vault unit.

For a single unit requiring three keys:

.. code-block:: none

   export VAULT_CACERT="/home/ubuntu/tls/ca.crt"
   export VAULT_ADDR="https://vault.example.com:8200"

   vault operator unseal
   vault operator unseal
   vault operator unseal

For multiple vault units, repeat the procedure by using a different value each
time for ``VAULT_ADDR``.

For more information on unsealing Vault see cloud operation :doc:`Unseal Vault
<ops-unseal-vault>`.
