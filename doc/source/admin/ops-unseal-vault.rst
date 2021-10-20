:orphan:

============
Unseal Vault
============

Preamble
--------

The Vault service always starts in a sealed state. Unsealing is the process of
obtaining the master key necessary to read the decryption key that decrypts the
data stored within. Prior to unsealing, therefore, Vault cannot be accessed by
the cloud.

.. important::

   Unsealing involves the input of special unseal keys, the number of which
   depends on how Vault was originally initialised. Without these keys Vault
   cannot be unsealed.

Procedure
---------

.. note::

   Ensure that the ``vault`` snap is installed on your Juju client host. You
   will need it to manage the Vault that is deployed in your cloud.

The output to :command:`juju status vault` should show that Vault is sealed:

.. code-block:: console

   Unit      Workload  Agent  Machine  Public address  Ports     Message
   vault/0*  blocked   idle   3/lxd/3  10.0.0.204      8200/tcp  Unit is sealed

Unseal **each** vault unit.

.. note::

   If the Vault API is encrypted see cloud operation :doc:`Configure TLS for
   the Vault API <ops-config-tls-vault-api>`.

For a single unit requiring three keys (``vault/0`` with IP address
10.0.0.204):

.. code-block:: none

   export VAULT_ADDR="http://10.0.0.204:8200"

   vault operator unseal
   vault operator unseal
   vault operator unseal

You will be prompted for the unseal keys. The information will not be echoed
back to the screen nor captured in the shell's history.

The output to :command:`juju status vault` should eventually contain:

.. code-block:: console

   Unit      Workload  Agent  Machine  Public address  Ports     Message
   vault/0*  active    idle   0/lxd/0  10.0.0.204      8200/tcp  Unit is ready (active: true, mlock: disabled)

.. note::

   It can take a few minutes for the "ready" status to appear. To expedite,
   force a status update: ``juju run -u vault/0 hooks/update-status``.

For multiple vault units, repeat the procedure by using a different value each
time for ``VAULT_ADDR``. For a three-member Vault cluster the output should
look similar to:

.. code-block:: console

   Unit                  Workload  Agent  Machine  Public address  Ports     Message
   vault/0               active    idle   0/lxd/0  10.0.0.204      8200/tcp  Unit is ready (active: true, mlock: disabled)
     vault-hacluster/1   active    idle            10.0.0.204                Unit is ready and clustered
   vault/1*              active    idle   1/lxd/0  10.0.0.205      8200/tcp  Unit is ready (active: false, mlock: disabled)
     vault-hacluster/0*  active    idle            10.0.0.205                Unit is ready and clustered
   vault/2               active    idle   2/lxd/0  10.0.0.206      8200/tcp  Unit is ready (active: false, mlock: disabled)
     vault-hacluster/2   active    idle            10.0.0.206                Unit is ready and clustered
