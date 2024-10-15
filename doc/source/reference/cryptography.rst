============
Cryptography
============

TLS transport encryption
------------------------
All network endpoints exposed by OpenStack services are secured using TLS.
Charmed OpenStack allows the user to provide their own SSL certificate,
enabling the use of certificates issued by their preferred Certificate
Authority. Charmed OpenStack also uses legacy `Vault`_ which generates a private key
and certificate bundle directly, using RSA with 2048-bit keys. The internal
cryptographic functions within Vault are delegated to crypto.rsa as provided in
`Golang cryptography`_.

Virtual machine migrations over SSH
-----------------------------------
Charmed OpenStack uses the libvirt driver for live virtual machine migrations
between hosts. This migration between hypervisor units occurs over SSH,
secured using 2048-bit RSA SSH keys for authentication.

OVN: Network virtualization
---------------------------
OVN (Open Virtual Network) network virtualization is implemented in Charmed
OpenStack through the neutron-api-plugin-ovn, ovn-central, and ovn-chassis
charms and is secured through TLS. User authentication is implemented through
the use of client certificates ensuring only trusted components can participate
in the network.

Keystone: Authentication tokens
-------------------------------
Keystone (the OpenStack identity service) issues tokens upon successful
authentication, which are used to access other OpenStack services. Keystone
uses `Fernet`_ symmetric encryption tokens by default which consist of a 128-bit
AES key and a 128-bit SHA256 HMAC signing key. `Python's cryptography library`_
as distributed in Ubuntu provides the cryptographic functions necessary for 
token generation and validation.

Barbican: Secrets management
----------------------------
Barbican (the OpenStack key management service) provides secrets management for
OpenStack services and integrates with Vault for storage of secrets. Secrets
managed by Barbican can be used for encrypting volumes in Cinder or TLS private
keys for load balancers. Barbican does not perform any cryptographic operations
directly.

Vault: Secrets storage
----------------------
The Vault service used by Barbican stores and retrieves information using a
4096-bit RSA encryption key. Access to the encryption key is protected using a
root key which in turn is protected by multiple unseal keys providing a `Sharmir
seal`_, of which a configurable number (typically 3 out of 5) must be provided to
unseal the Vault deployment and provide the service access to the root key
which is then used to access the main encryption key. Vault uses the standard
`Golang cryptography`_ module for cryptographic operations.

Access to Vault is secured using TLS (see TLS transport encryption).

.. LINKS
.. _Python's cryptography library: https://cryptography.io/en/latest/
.. _Fernet: https://docs.openstack.org/keystone/latest/admin/fernet-token-faq.html
.. _Vault: https://developer.hashicorp.com/vault/docs
.. _Sharmir seal: https://developer.hashicorp.com/vault/docs/concepts/seal
.. _Golang cryptography: https://pkg.go.dev/crypto