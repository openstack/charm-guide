:orphan:

=========================================
Reissue TLS certificates across the cloud
=========================================

Preamble
--------

New certificates can be reissued to all cloud clients that are currently
TLS-enabled. This is easily done with an action available to the vault charm.

One use case for this operation is when a cloud's existing application
certificates have expired.

.. important::

   This operation may cause momentary downtime for all API services that are
   being issued new certificates. Plan for a short maintenance window of
   approximately 15 minutes, including post-operation verification tests.

Certificate inspection
----------------------

TLS certificates can be inspected with the :command:`openssl` command with
output compared before and after the operation. In these examples, the Glance
API is listening on 10.0.0.220:9292.

Examples:

a) Expiration dates:

.. code-block:: none

   echo | openssl s_client -showcerts -connect 10.0.0.220:9292 2>/dev/null \
      | openssl x509 -inform pem -noout -text | grep Validity -A2

Output:

.. code-block:: console

        Validity
            Not Before: Sep 24 20:19:38 2021 GMT
            Not After : Sep 24 19:20:08 2022 GMT

b) Certificate chain:

.. code-block:: none

   echo | openssl s_client -showcerts -connect 10.0.0.220:9292 2>/dev/null \
      | openssl x509 -inform pem -noout -text | sed -n '/-----BEGIN/,/-----END/p'

Output:

.. code-block:: console

   ----BEGIN CERTIFICATE-----
   MIIEPjCCAyagAwIBAgIUOkw3afcFa47rmYSGwdqphiboh5kwDQYJKoZIhvcNAQEL
   BQAwPTE7MDkGA1UEAxMyVmF1bHQgUm9vdCBDZXJ0aWZpY2F0ZSBBdXRob3JpdHkg
   KGNoYXJtLXBraS1sb2NhbCkwHhcNMjEwOTI0MjAxOTM4WhcNMjIwOTI0MTkyMDA4
   .
   .
   .
   jcfdFmuy6hSHaqaV3XN//nZlk7yRlmMOisGXVQFvrxWg5xyfc56353hC6FQ1tXre
   gXr20uy5HKUkNulJXhcqxqC2Txevs/KJG2TXc3oKrBManFdw0BHT3qoeK91GDdVO
   tSHFWJB+kc74RajveqYOjXiC20Ei+bJaQgwrviyPL8W1qQ==
   -----END CERTIFICATE-----

Procedure
---------

To reissue new certificates to all TLS-enabled clients run the
``reissue-certificates`` action on the leader unit:

.. code-block:: none

   juju run-action --wait vault/leader reissue-certificates

The output to the :command:`juju status` command for the model will show
activity for each affected service as their corresponding endpoints get updated
via hook calls, for example:

.. code-block:: console

   Unit                         Workload  Agent      Machine  Public address  Ports              Message
   ceph-mon/0                   active    idle       0/lxd/0  10.0.0.231                         Unit is ready and clustered
   ceph-mon/1                   active    idle       1/lxd/0  10.0.0.235                         Unit is ready and clustered
   ceph-mon/2*                  active    idle       2/lxd/0  10.0.0.217                         Unit is ready and clustered
   ceph-osd/0*                  active    idle       0        10.0.0.203                         Unit is ready (1 OSD)
   ceph-osd/1                   active    idle       1        10.0.0.216                         Unit is ready (1 OSD)
   ceph-osd/2                   active    idle       2        10.0.0.219                         Unit is ready (1 OSD)
   cinder/0*                    active    executing  1/lxd/1  10.0.0.230      8776/tcp           Unit is ready
     cinder-ceph/0*             active    idle                10.0.0.230                         Unit is ready
     cinder-mysql-router/0*     active    idle                10.0.0.230                         Unit is ready
   glance/0*                    active    executing  2/lxd/1  10.0.0.220      9292/tcp           Unit is ready
     glance-mysql-router/0*     active    idle                10.0.0.220                         Unit is ready
   keystone/0*                  active    executing  0/lxd/1  10.0.0.225      5000/tcp           Unit is ready
     keystone-mysql-router/0*   active    idle                10.0.0.225                         Unit is ready
   mysql-innodb-cluster/0       active    executing  0/lxd/2  10.0.0.240                         Unit is ready: Mode: R/O, Cluster is ONLINE and can tolerate up to ONE failure.
   mysql-innodb-cluster/1       active    executing  1/lxd/2  10.0.0.208                         Unit is ready: Mode: R/O, Cluster is ONLINE and can tolerate up to ONE failure.
   mysql-innodb-cluster/2*      active    executing  2/lxd/2  10.0.0.218                         Unit is ready: Mode: R/W, Cluster is ONLINE and can tolerate up to ONE failure.
   neutron-api/0*               active    idle       1/lxd/3  10.0.0.238      9696/tcp           Unit is ready
     neutron-api-plugin-ovn/0*  active    executing           10.0.0.238                         Unit is ready
     neutron-mysql-router/0*    active    idle                10.0.0.238                         Unit is ready
   nova-cloud-controller/0*     active    executing  0/lxd/3  10.0.0.236      8774/tcp,8775/tcp  Unit is ready
     nova-mysql-router/0*       active    idle                10.0.0.236                         Unit is ready
   nova-compute/0*              active    idle       0        10.0.0.203                         Unit is ready
     ntp/0*                     active    idle                10.0.0.203      123/udp            chrony: Ready
     ovn-chassis/0*             active    executing           10.0.0.203                         Unit is ready
   ovn-central/0                active    executing  0/lxd/4  10.0.0.228      6641/tcp,6642/tcp  Unit is ready (northd: active)
   ovn-central/1                active    executing  1/lxd/4  10.0.0.232      6641/tcp,6642/tcp  Unit is ready
   ovn-central/2*               active    executing  2/lxd/3  10.0.0.213      6641/tcp,6642/tcp  Unit is ready (leader: ovnnb_db, ovnsb_db)
   placement/0*                 active    executing  2/lxd/4  10.0.0.210      8778/tcp           Unit is ready
     placement-mysql-router/0*  active    idle                10.0.0.210                         Unit is ready
   rabbitmq-server/0*           active    idle       2/lxd/5  10.0.0.206      5672/tcp           Unit is ready
   vault/0*                     active    idle       0/lxd/5  10.0.0.227      8200/tcp           Unit is ready (active: true, mlock: disabled)
     vault-mysql-router/0*      active    idle                10.0.0.227                         Unit is ready

Verification
------------

Verify that cloud service endpoints are available and are using HTTPS:

.. code-block:: none

   openstack endpoint list

Sample output:

.. code-block:: console

   ----------------------------------+-----------+--------------+--------------+---------+-----------+------------------------------+
   | ID                               | Region    | Service Name | Service Type | Enabled | Interface | URL                          |
   +----------------------------------+-----------+--------------+--------------+---------+-----------+------------------------------+
   | 181cc040c4c141d78a0f942dd584ac22 | RegionOne | keystone     | identity     | True    | public    | https://10.0.0.225:5000/v3   |
   | 235bd5e3831443afb4bf46929d1840c8 | RegionOne | placement    | placement    | True    | public    | https://10.0.0.210:8778      |
   | 2dd78e0f745b4bd49f92256d95187a30 | RegionOne | keystone     | identity     | True    | admin     | https://10.0.0.225:35357/v3  |
   | 39773c0683da4a0bb60909c12e7db69a | RegionOne | nova         | compute      | True    | public    | https://10.0.0.203:8774/v2.1 |
   | 49e72a65aa2f441db8e78e641bf6fe0c | RegionOne | placement    | placement    | True    | admin     | https://10.0.0.210:8778      |
   | 566e4d3850c64da38274e53a556eebe9 | RegionOne | neutron      | network      | True    | public    | https://10.0.0.238:9696      |
   | 7a803410e3344ce6912b7124b486ef4a | RegionOne | nova         | compute      | True    | admin     | https://10.0.0.203:8774/v2.1 |
   | 823c22a4951549169714d9e368dfe760 | RegionOne | nova         | compute      | True    | internal  | https://10.0.0.203:8774/v2.1 |
   | 9231f55f7d23442a9915a4321c3fc0e8 | RegionOne | placement    | placement    | True    | internal  | https://10.0.0.210:8778      |
   | b0e384c7368f4110b770eb56c3d720e1 | RegionOne | neutron      | network      | True    | internal  | https://10.0.0.238:9696      |
   | c658bd5a200d4111a31ae71e31503c35 | RegionOne | glance       | image        | True    | public    | https://10.0.0.220:9292      |
   | ce49bdeb066b4e3bafa97eec7cfec657 | RegionOne | glance       | image        | True    | internal  | https://10.0.0.220:9292      |
   | d320d4fc76574d2b806a8e88152b4ea1 | RegionOne | keystone     | identity     | True    | internal  | https://10.0.0.225:5000/v3   |
   | e6676dbb9e784e8880c00f6fbc8dd4b6 | RegionOne | glance       | image        | True    | admin     | https://10.0.0.220:9292      |
   | ec5d565e34124cdd8e694aaef8705611 | RegionOne | neutron      | network      | True    | admin     | https://10.0.0.238:9696      |
   +----------------------------------+-----------+--------------+--------------+---------+-----------+------------------------------+

Also check the successful resumption of cloud operations by running a routine
battery of tests. The creation of a VM is a good choice.
