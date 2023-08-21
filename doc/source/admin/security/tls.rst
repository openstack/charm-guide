=========================
Managing TLS certificates
=========================

.. important::

   This page has been identified as being affected by the breaking changes
   introduced between versions 2.9.x and 3.x of the Juju client. Read
   support note :ref:`juju_29_3x_changes` before continuing.

Overview
--------

The encryption of API endpoints in an OpenStack cloud requires a method for the
creation and distribution of TLS certificates, as well as the management of a
certificate authority (CA). Charmed OpenStack supports two ways of doing this:

#. on a per-model basis (using the vault application)
#. on a per-application basis (using charm configuration options)

.. note::

   The recommended way to manage TLS in Charmed OpenStack is with Vault. It is
   a centrally managed encryption solution, it is designed for the task, and it
   integrates well with charms.

Certificate management with Vault
---------------------------------

First ensure that Vault is deployed to the model containing the cloud and the
required post-deployment steps have been completed. See the `vault`_ charm
README for instructions.

The next step is to add a CA certificate to the model.

.. _add_ca_certificate:

Add a CA certificate
~~~~~~~~~~~~~~~~~~~~

For Vault to be able to issue certificates on your behalf you must equip it
with a CA certificate. This is done in **one** of two ways:

#. a Vault-generated self-signed root CA certificate
#. a third-party intermediate CA certificate

The Vault method is by far the simpler of the two.

Self-signed root CA certificate
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To have Vault generate a self-signed root CA certificate:

.. code-block:: none

   juju run --wait vault/leader generate-root-ca

You're done.

Third-party intermediate CA certificate
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The supported workflow for a third-party certificate involves three steps:

#. retrieve a Certificate Signing Request (CSR) for an intermediate CA from
   Vault
#. transfer the CSR to an external CA to have a signed certificate created
#. upload certificate information to Vault

Retrieve a CSR from Vault
.........................

To retrieve a Vault-generated CSR run the ``get-csr`` action on the leader
unit:

.. code-block:: none

   juju run --wait vault/leader get-csr

.. note::

   The CSR may be rejected by the signing CA due to incorrect values for
   certain properties (e.g. common-name). It would be prudent therefore to
   inquire before generating the CSR. Use command :command:`juju actions
   --schema vault` for help on setting properties with action ``get-csr``.

This will produce output similar to:

.. code-block:: console

   unit-vault-0:
     UnitId: vault/0
     id: "8"
     results:
       Stdout: |
         lxc
         lxc
         active
         lxc
       output: |-
         -----BEGIN CERTIFICATE REQUEST-----
         MIICijCCAXICAQAwRTFDMEEGA1UEAxM6VmF1bHQgSW50ZXJtZWRpYXRlIENlcnRp
         ZmljYXRlIEF1dGhvcml0eSAoY2hhcm0tcGtpLWxvY2FsKTCCASIwDQYJKoZIhvcN
         AQEBBQADggEPADCCAQoCggEBAJvof3Gut71YY9Ke3TlAYT+AoVUu8w0q2DKGh7dL
         5mUggcvThZuckbuj8IJZZ3pl5D114REcRRH9DIRxp4tH0TSmnb0PJLdjnuLyMQqy
         /IEipmSQWiILBF8c/QjYqEkvUoprADeJ+9L9KGc/axwuIoLWHqaXLnkSFzypgyz+
         9Qvxir4wSPvyygZVUDJvUoEekk/sMBidzpEaKuMF7U+aZAdlZvEPr39FilEwcUgQ
         EY2m3bDDe5maNcD6+la95ENuo0kuHF6wkjXuLGkzDV5xYBMtSO8sqymwRA1CPLyr
         WIA+ciDQ11Hy+1Q+YTurOoWzmr48QPlamCZEIz8BZeuf8vsCAwEAAaAAMA0GCSqG
         SIb3DQEBCwUAA4IBAQAoPhk5k5nXpFSYfbsOvm8Rc0hHUTfEHgB4xcQfzMrMTDMX
         fVmiJjGQhiM1q+eKNLLTDxuOGBBbyniQsveV6JZwpOlkOZ1YVdkw0EoaQndz6dEA
         JNjjelV2z1FxppKT3504uX/YkASTDnpb63aknE4W3C5aZSvyx/qw/WdUauCNnYoV
         NdFrzy0p2qm8kXwPsbjIwZTq/AqQ4t7UrNoXoONcxjAdq5UpuoBxgbRJJ7zr1RJp
         NUhVk/1qi9EQSGeigkuzGGPeRdBXvw4NXAXwnQfCiIBHgLEfkE3PVHNbXfVYqtjC
         3D2eeYPraKcSJIEts4DCJnbhj5FEzi1km9QgSZgA
         -----END CERTIFICATE REQUEST-----
     status: completed
     timing:
       completed: 2021-02-16 22:40:12 +0000 UTC
       enqueued: 2021-02-16 22:40:08 +0000 UTC
       started: 2021-02-16 22:40:09 +0000 UTC

Place the CSR data (minus any leading whitespace) in a file, say
``~/csr_file``. In this example, the file's contents would be:

.. code-block:: console

   -----BEGIN CERTIFICATE REQUEST-----
   MIICijCCAXICAQAwRTFDMEEGA1UEAxM6VmF1bHQgSW50ZXJtZWRpYXRlIENlcnRp
   ZmljYXRlIEF1dGhvcml0eSAoY2hhcm0tcGtpLWxvY2FsKTCCASIwDQYJKoZIhvcN
   AQEBBQADggEPADCCAQoCggEBAJvof3Gut71YY9Ke3TlAYT+AoVUu8w0q2DKGh7dL
   5mUggcvThZuckbuj8IJZZ3pl5D114REcRRH9DIRxp4tH0TSmnb0PJLdjnuLyMQqy
   /IEipmSQWiILBF8c/QjYqEkvUoprADeJ+9L9KGc/axwuIoLWHqaXLnkSFzypgyz+
   9Qvxir4wSPvyygZVUDJvUoEekk/sMBidzpEaKuMF7U+aZAdlZvEPr39FilEwcUgQ
   EY2m3bDDe5maNcD6+la95ENuo0kuHF6wkjXuLGkzDV5xYBMtSO8sqymwRA1CPLyr
   WIA+ciDQ11Hy+1Q+YTurOoWzmr48QPlamCZEIz8BZeuf8vsCAwEAAaAAMA0GCSqG
   SIb3DQEBCwUAA4IBAQAoPhk5k5nXpFSYfbsOvm8Rc0hHUTfEHgB4xcQfzMrMTDMX
   fVmiJjGQhiM1q+eKNLLTDxuOGBBbyniQsveV6JZwpOlkOZ1YVdkw0EoaQndz6dEA
   JNjjelV2z1FxppKT3504uX/YkASTDnpb63aknE4W3C5aZSvyx/qw/WdUauCNnYoV
   NdFrzy0p2qm8kXwPsbjIwZTq/AqQ4t7UrNoXoONcxjAdq5UpuoBxgbRJJ7zr1RJp
   NUhVk/1qi9EQSGeigkuzGGPeRdBXvw4NXAXwnQfCiIBHgLEfkE3PVHNbXfVYqtjC
   3D2eeYPraKcSJIEts4DCJnbhj5FEzi1km9QgSZgA
   -----END CERTIFICATE REQUEST-----

Have the CSR signed
...................

The procedure for obtaining a signed certificate from an external CA is
particular to the given CA, but it always entails sending the CSR to the CA
(typically from its website) and waiting for a reply.

For informational purposes, an example CLI command is provided below. The exact
command syntax is dependent upon the CA. Note the inclusion of the input file
``~/csr_file``:

.. code-block:: none

   openssl ca -config openssl.cnf -extensions v3_intermediate_ca -days 3650 \
      -notext -md sha256 -in ~/csr_file -out ~/vault-charm-int.pem -batch \
      -passin pass:secretpassword

.. note::

   Depending on the services deployed on your cloud, the intermediate CA
   may need to issue both client and server certificates. Therefore, it is
   necessary to check the property of your external CA and grant the
   intermediate CA the proper level of authorization to issue both types
   of certificates.

The certificate is normally provided in PEM format, like the output file
``~/vault-charm-int.pem`` in the above command. A root CA certificate should
also be provided and placed in, say, file ``~/root-ca.pem``.

Upload the signed certificate and the root CA certificate to Vault
..................................................................

To upload certificate information to Vault run the ``upload-signed-csr``
action on the leader unit:

.. code-block:: none

   juju run --wait vault/leader upload-signed-csr \
       pem="$(cat ~/vault-charm-int.pem | base64)" \
       root-ca="$(cat ~/root-ca.pem | base64)" \
       allowed-domains='openstack.local'

The file that the ``pem`` parameter refers to must contain a PEM bundle
consisting of:

#. the signed certificate
#. any other intermediate CA certificates
#. the root CA certificate

The file that the ``root-ca`` parameter refers to must contain a PEM bundle
consisting of:

#. any other intermediate CA certificates
#. the root CA certificate

.. important::

   Omitting the (other) intermediate certificate information will result in the
   new certificate being rejected (due to an incomplete trust chain).

See the following resources:

* `RFC5280`_: for details concerning certificate paths and trust
* `RFC7468`_: for details on the format of certificate PEM bundles

Issuing of certificates
~~~~~~~~~~~~~~~~~~~~~~~

Now that Vault is in possession of a CA certificate it will be able to issue
certificates to clients (API services). Client requests are made via the
``vault:certificates`` relation. For example:

.. code-block:: none

   juju integrate keystone:certificates vault:certificates
   juju integrate nova-cloud-controller:certificates vault:certificates
   juju integrate cinder:certificates vault:certificates
   juju integrate neutron-api:certificates vault:certificates
   juju integrate glance:certificates vault:certificates

A request will result in the transfer of certificates and keys from Vault. The
corresponding API endpoint will also be updated in Keystone's service catalogue
list to reflect that it is now using HTTPS. The service is now TLS-enabled.

A special client is mysql-innodb-cluster, the cloud database. It has a
self-signed certificate but it is recommended to use the one signed by Vault's
CA:

.. code-block:: none

   juju integrate mysql-innodb-cluster:certificates vault:certificates

.. important::

   Once Keystone is TLS-enabled every application that talks to Keystone (i.e.
   there exists a relation between the two) **must** be in possession of the
   CA certificate. This is achieved as a side-effect when enabling TLS for that
   application.

Verification
~~~~~~~~~~~~

To verify the CA certificate begin by sourcing the cloud init file and
inspecting the certificate's location and the Keystone API endpoint. The latter
should be using HTTPS:

.. code-block:: none

   source novarc
   env | grep -e OS_AUTH_URL -e OS_CACERT

Sample output is:

.. code-block:: console

   OS_CACERT=/home/ubuntu/snap/openstackclients/common/root-ca.crt
   OS_AUTH_URL=https://10.0.0.215:5000/v3

API services can now be queried by referring explicitly to the certificate. The
below tests correspond to the clients mentioned in the previous section:

.. code-block:: none

   # Keystone
   openstack --os-cacert $OS_CACERT catalog list
   # Nova
   openstack --os-cacert $OS_CACERT server list
   # Cinder
   openstack --os-cacert $OS_CACERT volume list
   # Neutron
   openstack --os-cacert $OS_CACERT network list
   # Glance
   openstack --os-cacert $OS_CACERT image list

Reissuing of certificates
~~~~~~~~~~~~~~~~~~~~~~~~~

New certificates can be reissued to all TLS-enabled clients by means of the
``reissue-certificates`` action. See cloud operation
:doc:`../ops-reissue-tls-certs` for details.

Switching between different types of CA certificates
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

It is possible to switch between a self-signed root CA certificate
and a third-party intermediate CA certificate after deployment.

.. important::

   Switching certificates will cause a short period of downtime for services
   using Vault as the certificate manager. Notably, a TLS-enabled Keystone will
   temporarily move to the ``maintainance`` state to update its endpoints.

From self-signed root certificate to third-party intermediate certificate
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To switch from a self-signed root CA certificate to a third-party intermediate
CA certificate, you need to first disable the PKI secrets
backend:

.. code-block:: none

   juju run --wait vault/leader disable-pki

This step deletes the existing root certificate and invalidates any previous
CSR requests.

Next, follow the steps described in the
`Third-party intermediate CA certificate`_ section above to retrieve a CSR
from the Vault and have it signed by the external CA.


Finally, upload both the signed intermediate certificate and the external root
CA certificate to Vault:

.. code-block:: none

   juju run --wait vault/leader upload-signed-csr \
      pem=“$(cat /path/to/vault-charm-int.pem | base64)" \
      root-ca="$(cat /path/to/root-ca.pem | base64)"

From third-party intermediate certificate to self-signed root certificate
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To switch from an external certificate to a self-signed one first disable the
PKI secrets backend and then generate a root CA certificate:

.. code-block:: none

   juju run --wait vault/leader disable-pki
   juju run --wait vault/leader generate-root-ca

Configuring SSL certificates via charm options
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. important::

   Updating a charm's SSL settings will change its status
   to ``maintenance``. The service will be temporarily unavailable during this
   short time.

Some OpenStack charms, such as `cinder`_, provide configuration options for
specifying a service certificate directly. This allows one to manage
certificates on a per-application basis.

Taking Cinder as an example, upload the entire certificate chain (
concatenated in the order of service certificate, chain of intermediate
certificates if available, and root CA certificate) by using the ``ssl_cert``
configuration option:

.. code-block:: none

   juju config cinder ssl_cert="$(cat /path/to/cert.pem \
      /path/to/intermediate.pem /path/to/myCA.pem| base64)"

.. note::

   Uploading the full certificate chain is recommended in all cases for best
   practices. It is especially required if the service uses Apache2 to
   communicate with public SSL clients. Nonetheless, if the service is used
   strictly internally and its certificate was issued directly by a root CA,
   uploading solely the service certificate is acceptable.

Next, upload the SSL key associated with the service certificate by using the
``ssl_key`` option:

.. code-block:: none

   juju config cinder ssl_key="$(cat /path/to/key.pem | base64)"

In the case that the service certificate is privately signed, its root CA
certificate should be uploaded to ``ssl_ca`` to enable system-wise
authorization:

.. code-block:: none

   juju config cinder ssl_ca="$(cat /path/to/myCA.pem | base64)"

.. LINKS
.. _RFC5280: https://tools.ietf.org/html/rfc5280#section-3.2
.. _RFC7468: https://tools.ietf.org/html/rfc7468#section-5
.. _vault: https://opendev.org/openstack/charm-vault/src/branch/master/src/README.md
.. _cinder: https://charmhub.io/cinder/
