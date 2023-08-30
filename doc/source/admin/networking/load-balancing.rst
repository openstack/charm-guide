==============
Load balancing
==============

.. important::

   This page has been identified as being affected by the breaking changes
   introduced between versions 2.9.x and 3.x of the Juju client. Read
   support note :ref:`juju_29_3x_changes` before continuing.

Overview
--------

OpenStack Octavia can be deployed to provide Load balancing services as part of
an OpenStack cloud. This service supersedes the LBaaS v2 services provided
directly through Neutron in earlier releases; when Octavia is deployed a proxy
service is configured to proxy LBaaS v2 API calls directly to Octavia.

.. note::

   Octavia is supported by Charmed OpenStack starting with OpenStack Rocky.

Octavia uses cloud resources to provision instances to provide LBaaS services
unlike the LBaaS v2 support in the OpenStack Charms which placed haproxy
instances on neutron-gateway units.

.. warning::

   There is no automatic migration path for Neutron LBaaS haproxy-on-host
   configurations (as deployed under the existing support) to Octavia Amphora
   configurations. New load balancers must be created in Octavia before the
   octavia charm is related to the existing neutron-api charm. Floating IPs
   can then be moved prior to deletion of existing LBaaS based balancers.

Deployment
----------

.. note::

   Throughout the deployment, ensure that the value for ``openstack-origin``
   matches the currently deployed OpenStack release.

Octavia uses OpenStack Barbican to store certificates for TLS termination on
load balancers. Barbican, in turn, uses Vault to securely store that data.

.. note::

   For Vault deployment instructions see the `vault charm`_. For certificate
   management information read the :doc:`../security/tls` page.

To deploy Barbican:

.. code-block:: none

   juju deploy barbican --config openstack-origin=cloud:bionic-rocky
   juju deploy barbican-vault
   juju integrate barbican mysql
   juju integrate barbican rabbitmq-server
   juju integrate barbican keystone
   juju integrate barbican barbican-vault
   juju integrate barbican-vault vault

Octavia can then be deployed. Use the appropriate section depending on your
cloud's networking framework:

Neutron ML2+OVS
~~~~~~~~~~~~~~~

.. code-block:: none

   juju deploy octavia --config openstack-origin=cloud:bionic-rocky
   juju integrate octavia rabbitmq-server
   juju integrate octavia mysql
   juju integrate octavia keystone
   juju integrate octavia neutron-openvswitch
   juju integrate octavia neutron-api
   juju config neutron-api enable-ml2-port-security=True

   juju deploy octavia-dashboard
   juju integrate octavia-dashboard openstack-dashboard

Neutron ML2+OVN
~~~~~~~~~~~~~~~

.. code-block:: none

   juju deploy octavia --config openstack-origin=cloud:bionic-ussuri
   juju integrate octavia rabbitmq-server
   juju integrate octavia mysql
   juju integrate octavia keystone
   juju integrate octavia:ovsdb-subordinate ovn-chassis:ovsdb-subordinate
   juju integrate octavia neutron-api
   juju config neutron-api enable-ml2-port-security=True

   juju deploy octavia-dashboard
   juju integrate octavia-dashboard openstack-dashboard

.. note::

   Octavia uses a Neutron network for communication between Octavia control
   plane services and Octavia Amphorae; units will deploy into a 'blocked'
   state until the configuration steps are executed.

Configuration
-------------

Generate certificates
~~~~~~~~~~~~~~~~~~~~~

Octavia uses client certificates for authentication and security of
communication between Amphorae (load balancers) and the Octavia control plane.

The commands below show how keys and certificates can be generated. These are
examples only; modify the parameters as required.

.. code-block:: none

   mkdir -p demoCA/newcerts
   touch demoCA/index.txt
   touch demoCA/index.txt.attr

   openssl genpkey -algorithm RSA -aes256 -pass pass:foobar -out issuing_ca_key.pem
   openssl req -x509 -passin pass:foobar -new -nodes -key issuing_ca_key.pem \
       -config /etc/ssl/openssl.cnf \
       -subj "/C=US/ST=Somestate/O=Org/CN=www.example.com" \
       -days 365 \
       -out issuing_ca.pem

   openssl genpkey -algorithm RSA -aes256 -pass pass:foobar -out controller_ca_key.pem
   openssl req -x509 -passin pass:foobar -new -nodes \
           -key controller_ca_key.pem \
       -config /etc/ssl/openssl.cnf \
       -subj "/C=US/ST=Somestate/O=Org/CN=www.example.com" \
       -days 365 \
       -out controller_ca.pem
   openssl req \
       -newkey rsa:2048 -nodes -keyout controller_key.pem \
       -subj "/C=US/ST=Somestate/O=Org/CN=www.example.com" \
       -out controller.csr
   openssl ca -passin pass:foobar -config /etc/ssl/openssl.cnf \
       -cert controller_ca.pem -keyfile controller_ca_key.pem \
       -create_serial -batch \
       -in controller.csr -days 365 -out controller_cert.pem
   cat controller_cert.pem controller_key.pem > controller_cert_bundle.pem

This information is then provided to Octavia via charm configuration options:

.. code-block:: none

   juju config octavia \
       lb-mgmt-issuing-cacert="$(base64 issuing_ca.pem)" \
       lb-mgmt-issuing-ca-private-key="$(base64 issuing_ca_key.pem)" \
       lb-mgmt-issuing-ca-key-passphrase=foobar \
       lb-mgmt-controller-cacert="$(base64 controller_ca.pem)" \
       lb-mgmt-controller-cert="$(base64 controller_cert_bundle.pem)"

Resource configuration
~~~~~~~~~~~~~~~~~~~~~~

The charm will automatically create and maintain the resources required for
operation of the Octavia service by running the `configure-resources` action
on the lead octavia unit:

.. code-block:: none

   juju run --wait octavia/0 configure-resources

This action must be run before Octavia is fully operational.

Access to the Octavia load-balancer API is guarded by policies and end users
must have specific roles to gain access to the service.  The charm will request
Keystone to pre-create these roles for you on deployment but you must assign the
roles to your end users as you see fit.  Take a look at
`Octavia Policies`_.

The charm also allows the operator to pre-configure these resources to support
full custom configuration of the management network for Octavia. If you want
to manage these resources yourself you must set the `create-mgmt-network`
configuration option to false.

Network resources for use by Octavia must be tagged using Neutron resource tags
(typically by passing a '--tag' CLI parameter when creating resources - see the
OpenStack CLI for more details) using the following schema:

=========================== ====================== =========================================================
Resource Type               Tag                    Description
=========================== ====================== =========================================================
Neutron Network             charm-octavia          Management network
Neutron Subnet              charm-octavia          Management network subnet
Neutron Router              charm-octavia          (Optional) Router for IPv6 RA or north/south mgmt traffic
Amphora Security Group      charm-octavia          Security group for Amphora ports
Controller Security Group   charm-octavia-health   Security group for Controller ports
=========================== ====================== =========================================================

Execution of the `configure-resources` action will detect the pre-configured
network resources in Neutron using tags and configure the Octavia service as
appropriate.

The UUID of the Nova flavor to use for Amphorae can be set using the
`custom-amp-flavor-id` configuration option.

Amphora image
~~~~~~~~~~~~~

Octavia uses Amphorae (cloud instances running HAProxy) to provide LBaaS
services; an appropriate image must be uploaded to Glance with the tag
`octavia-amphora`.

You can use the ``octavia-diskimage-retrofit`` tool to transform a stock Ubuntu
cloud image into a Octavia HAProxy Amphora image.

This tool is available as a snap and for convenience there is also a charm
available that can transform Ubuntu images already available in your Glance
image store.

Example usage:

.. code-block:: none

   juju deploy glance-simplestreams-sync
   juju deploy octavia-diskimage-retrofit \
       --config amp-image-tag=octavia-amphora

   juju integrate glance-simplestreams-sync keystone
   juju integrate glance-simplestreams-sync:certificates vault:certificates
   juju integrate octavia-diskimage-retrofit glance-simplestreams-sync
   juju integrate octavia-diskimage-retrofit keystone

After the deployment has settled and ``glance-simplestreams-sync`` has
completed its initial image sync, you may ask a ``octavia-diskimage-retrofit``
unit to initiate the Amphora image retrofitting process.

This is accomplished by running an action on one of the units.

.. code-block:: none

   juju run --wait octavia-diskimage-retrofit/leader retrofit-image

Octavia will use this image for all Amphora instances.

.. warning::

   It's important to keep the Amphora image up-to-date to ensure that LBaaS
   services remain secure; this process is not covered in this document.

   See the Octavia `operators maintenance`_ guide for more details.

Octavia user roles
------------------

To provide access to the Octavia API endpoints a load-balancer role must be
added to a user. For example:

.. code-block:: none

   openstack role add --user-domain admin_domain --user admin \
      --project-domain admin_domain --project admin \
      load-balancer_admin

See `Managing Octavia User Roles`_ in the upstream documentation.

.. note::

   Explicit user role assignments are required starting with OpenStack Wallaby.

Usage
-----

To deploy a basic HTTP load balancer using a floating IP for access:

.. code-block:: none

   lb_vip_port_id=$(openstack loadbalancer create -f value -c vip_port_id --name lb1 --vip-subnet-id private_subnet)

   # Re-run the following until lb1 shows ACTIVE and ONLINE status':
   openstack loadbalancer show lb1

   openstack loadbalancer listener create --name listener1 --protocol HTTP --protocol-port 80 lb1
   openstack loadbalancer pool create --name pool1 --lb-algorithm ROUND_ROBIN --listener listener1 --protocol HTTP
   openstack loadbalancer healthmonitor create --delay 5 --max-retries 4 --timeout 10 --type HTTP --url-path /healthcheck pool1
   openstack loadbalancer member create --subnet-id private_subnet --address 192.168.21.100 --protocol-port 80 pool1
   openstack loadbalancer member create --subnet-id private_subnet --address 192.168.21.101 --protocol-port 80 pool1

   floating_ip=$(openstack floating ip create -f value -c floating_ip_address ext_net)
   openstack floating ip set --port $lb_vip_port_id $floating_ip

The example above assumes:

* The user and project executing the example has a subnet configured with the
  name 'private_subnet' with the CIDR 192.168.21.0/24

* An external network definition for floating IPs has been configured by the
  cloud operator with the name 'ext_net'

* Two instances running HTTP services attached to 'private_subnet' on IP
  addresses 192.168.21.{100,101} exposing a heat check on '/healthcheck'

The example is also most applicable in cloud deployments that use overlay
networking for project networks and floating IPs for network ingress to project
networks.

For more information on creating and configuring load balancing services in
Octavia please refer to the `Octavia cookbook`_.

.. LINKS
.. _Octavia Policies: https://docs.openstack.org/octavia/latest/configuration/policy.html
.. _Octavia cookbook: https://docs.openstack.org/octavia/latest/user/guides/basic-cookbook.html
.. _operators maintenance: https://docs.openstack.org/octavia/latest/admin/guides/operator-maintenance.html#rotating-the-amphora-images
.. _vault charm: https://charmhub.io/vault/
.. _Managing Octavia User Roles: https://docs.openstack.org/octavia/latest/configuration/policy.html#managing-octavia-user-roles
