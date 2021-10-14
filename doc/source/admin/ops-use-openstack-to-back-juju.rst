:orphan:

=========================================
Use OpenStack as a backing cloud for Juju
=========================================

Preamble
--------

An OpenStack cloud can be used as a backing cloud to Juju. This means that
Juju-managed workloads will run in OpenStack VMs.

Requirements
------------

The glance-simplestreams-sync application will need to be deployed in the
OpenStack cloud. This will manage image downloads and place SimpleStreams
image metadata in Object Storage for Juju to consult in order to provision its
machines with those images.

Cloud operation :doc:`Implement automatic Glance image updates
<ops-auto-glance-image-updates>` has full deployment instructions.

Once the above requirement is met, the metadata should be available to Juju via
the ``image-stream`` endpoint, which points to an Object Storage URL. For
example:

.. code-block:: none

   openstack endpoint list --service image-stream

   +---------------------+-----------+--------------+-----------------+---------+-----------+-----------------------------------------------------+
   | ID                  | Region    | Service Name | Service Type    | Enabled | Interface | URL                                                 |
   +---------------------+-----------+--------------+-----------------+---------+-----------+-----------------------------------------------------+
   | 043b73545d804457... | RegionOne | image-stream | product-streams | True    | admin     | https://10.0.0.224:443/swift/simplestreams/data/    |
   | ad06281ba76e4cdf... | RegionOne | image-stream | product-streams | True    | public    | https://10.0.0.224:443/swift/v1/simplestreams/data/ |
   | e1baedce6e004da8... | RegionOne | image-stream | product-streams | True    | internal  | https://10.0.0.224:443/swift/v1/simplestreams/data/ |
   +---------------------+-----------+--------------+-----------------+---------+-----------+-----------------------------------------------------+

.. note::

   If the cloud is TLS-enabled, the initial image sync (whether manual or
   automatic) will update the ``image-stream`` endpoint to HTTPS.

Procedure
---------

The procedure will consist of adding a TLS-enabled OpenStack cloud to Juju,
adding a credential to Juju, and finally creating a Juju controller.

Add the cloud
~~~~~~~~~~~~~

Adding the cloud can be done either interactively (user prompts) or via a YAML
file.

Here we'll use file ``mystack-cloud.yaml`` to define a cloud called 'mystack':

.. code-block:: yaml

   clouds:
     mystack:
       type: openstack
       auth-types: userpass
       regions:
         RegionOne:
           endpoint: https://10.0.0.225:5000/v3
       ca-certificates:
       - |
         -----BEGIN CERTIFICATE-----
         MIIDazCCAlOgAwIBAgIUQ0ASDlfq4sWpPwrxjspBZ4DO+bgwDQYJKoZIhvcNAQEL
         BQAwPTE7MDkGA1UEAxMyVmF1bHQgUm9vdCBDZXJ0aWZpY2F0ZSBBdXRob3JpdHkg
         KGNoYXJtLXBraS1sb2NhbCkwHhcNMjEwODAyMjI1MjUyWhcNMzEwNzMxMjE1MzIx
         WjA9MTswOQYDVQQDEzJWYXVsdCBSb290IENlcnRpZmljYXRlIEF1dGhvcml0eSAo
         Y2hhcm0tcGtpLWxvY2FsKTCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEB
         ALjXjvYtmVoAw15Zuub41vRabiSZe8GF64nKb0EZxN9/13dAINYhusBX+5CHxFUm
         qOSmktu8DtKUvqpaoTgAAJerugbW2Xzmj23T9rKk4y3zoVPpuMRozN8Riv8itBaw
         LKImxKeUetDWwhWEO7uX0+5K48Vg5hhiiGZJaHaVU1eSjSWnVKFGbExgv9PsS4Wt
         AnL3awWuR/3NulZTmNHqnwNfb+2DffdQVTH7UyuqlNNhTyQZQOlKY1DwtHZiAMLE
         rb1yoLpx6gR4JR8PuohTqu0MrWNqZLQnnIMc/Ty3kRrfSTgJslwFTGyzBNRIYt13
         PF3c51lDlLwszqW3NfBlpIUCAwEAAaNjMGEwDgYDVR0PAQH/BAQDAgEGMA8GA1Ud
         EwEB/wQFMAMBAf8wHQYDVR0OBBYEFA38OdAQaIVjor23NY7m5seQACduMB8GA1Ud
         IwQYMBaAFA38OdAQaIVjor23NY7m5seQACduMA0GCSqGSIb3DQEBCwUAA4IBAQCj
         D/bVi/t0t1B7HI4IuBS3oLwxvy098qEu1/UPmJ6EXgEWT7Q/2SZrvdWXr8FJHbk3
         Meu5N+Sn5mksXRWhl6E7DXWGyABkvAQUdGgF6gQxg80XbX1LW6G1mzto1QeaCHZl
         Yl04rZzt2P5ut/CMJn6PFI7GhkhwOsrWKx2+wxZaLHwNFuGiNUJp8mOl0sCPqq7i
         1CKzbDp12oW6enWvL6zzntHB0VY4wE6OBwghHiXJ2FHSwClQoEzKxR6Z+onpw8EJ
         3ZkLiYiEs0fljKcKdBtnjc/PiKIC29OAcGEDGEdy2YX4mH19fNTZoAGkIkLg6CuW
         bSOc6nke3F1sEtda0CbQ
         -----END CERTIFICATE-----

The endpoint (the cloud's public Keystone endpoint) and the region can be
obtained by running ``openstack endpoint list``.

Because this example cloud is using TLS, we need to pass the related CA
certificate. This can be gathered, if using Vault, by running ``juju run-action
--wait vault/leader get-root-ca``.

To add the cloud:

.. code-block:: none

   juju add-cloud --client mystack -f mystack-cloud.yaml

See `Adding an OpenStack cloud`_ in the Juju documentation for more general
information.

Add a credential
~~~~~~~~~~~~~~~~

A Juju credential, which represents a set of OpenStack credentials, needs to be
associated with the newly added cloud. This can be done either interactively
(user prompts) or via a YAML file.

Here we'll use file ``mystack-creds.yaml`` to define a credential called, say,
'operator':

.. code-block:: yaml

   credentials:
     mystack:
       operator:
         auth-type: userpass
         version: "3"
         password: Boh9wiahee8xah5l
         username: admin
         tenant-name: admin
         user-domain-name: admin_domain
         project-domain-name: admin_domain

.. tip::

   Many of the values placed in the YAML file can be obtained by sourcing the
   cloud admin's init file and reviewing the values for commonly used OpenStack
   environmental variables (e.g. ``source openrc && env | grep OS_``).

To add the credential for cloud 'mystack':

.. code-block:: none

   juju add-credential --client mystack -f mystack-creds.yaml

See `Adding an OpenStack credential`_ in the Juju documentation for further
guidance.

Create a controller
~~~~~~~~~~~~~~~~~~~

Create a Juju controller called, say, 'over-controller' on cloud 'mystack'
using credential 'admin':

.. code-block:: none

   juju bootstrap --credential admin \
      mystack over-controller

Often a cloud will provide access to its VMs only through floating IP addresses
on a public network. In such a case, add constraint ``allocate-public-ip``:

.. code-block:: none

   juju bootstrap --credential admin \
      --bootstrap-constraints allocate-public-ip=true \
      mystack over-controller

Inspect the new controller:

.. code-block:: none

   juju controllers

   Controller        Model      User   Access     Cloud/Region    Models  Nodes    HA  Version
   over-controller*  default    admin  superuser  mystack/RegionOne    2      1  none  2.9.10
   under-controller  openstack  admin  superuser  corpstack/corpstack  2      1  none  2.9.0

Controller 'under-controller' is managing the original OpenStack cloud.

See `Creating a Juju controller for OpenStack`_ in the Juju documentation for
further guidance.

.. LINKS
.. _Adding an OpenStack cloud: https://juju.is/docs/olm/openstack#heading--adding-an-openstack-cloud
.. _Adding an OpenStack credential: https://juju.is/docs/olm/openstack#heading--adding-an-openstack-credential
.. _Creating a Juju controller for OpenStack: https://juju.is/docs/olm/openstack#heading--creating-a-juju-controller-for-openstack
