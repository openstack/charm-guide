==================
Encryption at Rest
==================

Overview
++++++++

As of the 18.05 release, the OpenStack charms support encryption of data in three
key areas - local ephemeral instance storage for Nova instances, Ceph OSD block
devices and Swift Storage block devices.

The objective of this feature is to mitigate the risk of data compromise in the
event that disks or full servers are removed from data center deployments.

Encryption of underlying block devices is performed using dm-crypt with LUKS; key
management is provided by Vault, which provides secure encrypted storage of the
keys used for each block device with automatic sealing of secrets in the event
of reboot/restart of services.

The objective of this feature is to mitigate the risk of data compromise in the
event that disks or full servers are removed from data center deployments.

Vault
+++++

See the `vault charm`_.

Enabling Encryption
+++++++++++++++++++

Encryption is enable via configuration options on the nova-compute, swift-storage and
ceph-osd charms and a relation to the vault application:

.. code:: bash

    juju config swift-storage encrypt=true
    juju config nova-compute encrypt=true ephemeral-device=/dev/bcache2
    juju config ceph-osd osd-encrypt=true osd-encrypt-keymanager=vault
    juju add-relation swift-storage:secrets-storage vault:secrets
    juju add-relation nova-compute:secrets-storage vault:secrets
    juju add-relation ceph-osd:secrets-storage vault:secrets

.. note::

    Encryption is only enabled during the initial preparation of the underlying
    block devices by the charms; enabling these options post deployment will
    not enable encryption on existing in-use devices.  As a result its best to
    enable this options as part of an overlay bundle for during initial
    deployment.

Security Design Notes
+++++++++++++++++++++

Consuming application units access Vault using a Vault AppRole and associated policy
which is specific to each machine in the deployment; The AppRole enforces uses
of a secret id and access is only permitted from the configured network address
of the consuming unit.  The associated policy only allows the consuming unit to
store and retrieve secrets from a specific secrets back-end under a specific
sub path (in this case the hostname of the unit).

The secret id for the AppRole is retrieved out-of-band from Juju by the
consuming charm; a one-shot retrieval token is provided over the relation
from vault to each consuming application which is specific to each unit which
can be used to retrieve the actual secret id;  the token also has a limited ttl
(2 hours) and the call must originate from the configured network address of
the consuming unit.  The secret id is only ever visible to the consuming unit
and vault itself, providing an additional layer of protection for deployments.

LUKS encryption keys are never store on local disk; vaultlocker is used to encrypt
and store the key in vault, and to retrieve the key and open encrypted block
devices during boot.  Keys are only ever held in memory.

.. LINKS
.. _vault charm: https://jaas.ai/vault/
