===========================================
Setting up internal DNS resolution with OVN
===========================================

OVN supports Neutron internal DNS resolution. To configure this:

.. note::

   Please see the :doc:`index` page for general information on using OVN with
   Charmed OpenStack.

.. caution::

   At the time of this writing the internal DNS support does not include
   reverse lookup (PTR-records) of instance IP addresses, only forward lookup
   (A and AAAA-records) of instance names. This is tracked in `LP #1857026`_.

.. code::

   juju config neutron-api enable-ml2-dns=true
   juju config neutron-api dns-domain=openstack.example.
   juju config neutron-api-plugin-ovn dns-servers="1.1.1.1 8.8.8.8"

.. note::

   The value for the ``dns-domain`` configuration option must
   not be set to 'openstack.local.' as that will effectively disable the
   feature.

   It is also important to end the string with a '.' (dot).

When you set ``enable-ml2-dns`` to 'true' and set a value for ``dns-domain``,
Neutron will add details such as instance name and DNS domain name to each
individual Neutron port associated with instances. The OVN ML2 driver will
populate the ``DNS`` table of the Northbound and Southbound databases:

.. code::

   # ovn-sbctl list DNS
   _uuid               : 2e149fa8-d27f-4106-99f5-a08f60c443bf
   datapaths           : [b25ed99a-89f1-49cc-be51-d215aa6fb073]
   external_ids        : {dns_id="4c79807e-0755-4d17-b4bc-eb57b93bf78d"}

   records             : {"c-1"="192.0.2.239", "c-1.openstack.example"="192.0.2.239"}

On the chassis, OVN creates flow rules to redirect UDP port 53 packets (DNS)
to the local ``ovn-controller`` process:

.. code::

   cookie=0xdeaffed, duration=77.575s, table=22, n_packets=0, n_bytes=0, idle_age=77, priority=100,udp6,metadata=0x2,tp_dst=53 actions=controller(userdata=00.00.00.06.00.00.00.00.00.01.de.10.00.00.00.64,pause),resubmit(,23)
   cookie=0xdeaffed, duration=77.570s, table=22, n_packets=0, n_bytes=0, idle_age=77, priority=100,udp,metadata=0x2,tp_dst=53 actions=controller(userdata=00.00.00.06.00.00.00.00.00.01.de.10.00.00.00.64,pause),resubmit(,23)

The local ``ovn-controller`` process then decides if it should respond to the
DNS query directly or if it needs to be forwarded to the real DNS server.

.. BUGS
.. _LP #1857026: https://bugs.launchpad.net/ubuntu/+source/ovn/+bug/1857026
