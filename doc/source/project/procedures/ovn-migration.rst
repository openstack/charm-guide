================
Migration to OVN
================

Starting with OpenStack Ussuri, Charmed OpenStack recommends OVN as the cloud's
software defined networking framework (SDN). This page outlines the procedure
for migrating an existing non-OVN cloud to OVN. Technically, it describes how
to move from "Neutron ML2+OVS" to "Neutron ML2+OVN".

On a charm level, the migration entails replacing these charms:

* neutron-gateway
* neutron-openvswitch

with these charms:

* ovn-central
* ovn-chassis (or ovn-dedicated-chassis)
* neutron-api-plugin-ovn charms

Post-migration, the :doc:`../../admin/networking/ovn/index` page includes
information on configuration and usage.

MTU considerations
------------------

When migrating from ML2+OVS to ML2+OVN there will be a change of encapsulation
for the tunnels in the overlay network to ``geneve``. A side effect of the
change of encapsulation is that the packets transmitted on the physical network
get larger.

You must examine the existing configuration of network equipment, physical
links on hypervisors and configuration of existing virtual project networks to
determine if there is room for this growth.

Making room for the growth could be accomplished by increasing the MTU
configuration on the physical network equipment and hypervisor physical links.
If this can be done then steps #1 and #9 below can be skipped, where it is
shown how to **reduce** the MTU on all existing cloud instances.

Remember to take any other encapsulation used in your physical network
equipment into account when calculating the MTU (VLAN tags, MPLS labels etc.).

Encapsulation types and their overhead:

+---------------+----------+------------------------+
| Encapsulation | Overhead | Difference from Geneve |
+===============+==========+========================+
| Geneve        | 38 Bytes |                0 Bytes |
+---------------+----------+------------------------+
| VXLAN         | 30 Bytes |                8 Bytes |
+---------------+----------+------------------------+
| GRE           | 22 Bytes |               16 bytes |
+---------------+----------+------------------------+

Confirmation of migration actions
---------------------------------

Many of the actions used for the migration require a confirmation from the
operator by way of the ``i-really-mean-it`` parameter.

This parameter accepts the values 'true' or 'false'. If 'false' the requested
operation will either not be performed, or will be performed in dry-run mode,
if 'true' the requested operation will be performed.

In the examples below the parameter will not be listed, this is deliberate to
avoid accidents caused by cutting and pasting the wrong command into a
terminal.

Prepare for the migration
-------------------------

This section contains the preparation steps that will ensure minimal instance
down time during the migration. Ensure that you have studied them in advance
of the actual migration.

.. important::

   Allow for at least 24 hours to pass between the completion of the
   preparation steps and the commencement of the actual migration steps.
   This is particularly necesseary because depending on your physical network
   configuration, it may be required to reduce the MTU size on all cloud
   instances as part of the migration.

1. Reduce MTU on all instances in the cloud if required

   Please refer to the MTU considerations section above.

   * Instances using DHCP can be controlled centrally by the cloud operator
     by overriding the MTU advertised by the DHCP server.

     .. code-block:: none

         juju config neutron-gateway instance-mtu=1300

         juju config neutron-openvswitch instance-mtu=1300

   * Instances using IPv6 RA or SLAAC will automatically adjust
     their MTU as soon as OVN takes over announcing the RAs.

   * Any instances not using DHCP must be configured manually by the end user of
     the instance.

2. Confirm cloud subnet configuration

   * Confirm that all subnets have IP addresses available for allocation.

     During the migration OVN may create a new port in subnets and allocate an
     IP address to it. Depending on the type of network, this port will be used
     for either the OVN metadata service or for the SNAT address assigned to an
     external router interface.

     .. warning::

        If a subnet has no free IP addresses for allocation the migration will
        fail.

   * Confirm that all subnets have a valid DNS server configuration.

     OVN handles instance access to DNS differently to how ML2+OVS does. Please
     refer to the Internal DNS resolution paragraph in this document for
     details.

     When the subnet ``dns_nameservers`` attribute is empty the OVN DHCP server
     will provide instances with the DNS addresses specified in the
     neutron-api-plugin-ovn ``dns-servers`` configuration option. If any of
     your subnets have the ``dns_nameservers`` attribute set to the IP address
     ML2+OVS used for instance DNS (usually the .2 address of the project
     subnet) you will need to remove this configuration.

3. Make a fresh backup copy of the Neutron database

4. Deploy the OVN components and Vault

   In your Juju model you can have a charm deployed multiple times using
   different application names. In the text below this will be referred to as
   "named application". One example where this is common is for deployments
   with Octavia where it is common to use a separate named application for
   neutron-openvswtich for use with the Octavia units.

   In addition to the central components you should deploy an ovn-chassis
   named application for every neutron-openvswitch named application in your
   deployment. For every neutron-gateway named application you should deploy an
   ovn-dedicated-chassis named application to the same set of machines.

   At this point in time each hypervisor or gateway will have a Neutron
   Open vSwitch (OVS) agent managing the local OVS instance. Network loops
   may occur if an ovn-chassis unit is started as it will also attempt to
   manage OVS. To avoid this, deploy ovn-chassis (or ovn-dedicated-chassis) in
   a paused state by setting the ``new-units-paused`` configuration option to
   'true':

   .. code-block:: none

      juju deploy ovn-central \
         --series focal \
         -n 3 \
         --to lxd:0,lxd:1,lxd:2

      juju deploy ovn-chassis \
         --series focal \
         --config new-units-paused=true \
         --config bridge-interface-mappings='br-provider:00:00:5e:00:00:42' \
         --config ovn-bridge-mappings=physnet1:br-provider

      juju deploy ovn-dedicated-chassis \
         --series focal \
         --config new-units-paused=true \
         --config bridge-interface-mappings='br-provider:00:00:5e:00:00:51' \
         --config ovn-bridge-mappings=physnet1:br-provider \
         -n 2 \
         --to 3,4

      juju deploy --series focal mysql-router vault-mysql-router
      juju deploy --series focal vault

      juju add-relation vault-mysql-router:db-router \
         mysql-innodb-cluster:db-router
      juju add-relation vault-mysql-router:shared-db vault:shared-db

      juju add-relation ovn-central:certificates vault:certificates

      juju add-relation ovn-chassis:certificates vault:certificates
      juju add-relation ovn-chassis:ovsdb ovn-central:ovsdb
      juju add-relation nova-compute:neutron-plugin ovn-chassis:nova-compute

   The values to use for the ``bridge-interface-mappings`` and
   ``ovn-bridge-mappings`` configuration options can be found by looking at
   what is set for the ``data-port`` and ``bridge-mappings`` configuration
   options on the neutron-openvswitch and/or neutron-gateway applications.

   .. note::

      In the above example the placement given with the ``--to`` parameter to
      :command:`juju` is just an example. Your deployment may also have
      multiple named applications of the neutron-openvswitch charm and/or
      mutliple applications related to the neutron-openvswitch named
      applications. You must tailor the commands to fit with your deployments
      topology.

5. Unseal Vault (see the `vault charm`_), set up TLS certificates (see
   :doc:`../../admin/security/tls`), and validate that the services on
   ovn-central units are running as expected. Please refer to the OVN
   :doc:`../../admin/networking/ovn/index` page for more information.

Perform the migration
---------------------

6. Change firewall driver to 'openvswitch'

   To be able to successfully clean up after the Neutron agents on hypervisors
   we need to instruct the neutron-openvswitch charm to use the 'openvswitch'
   firewall driver. This is accomplished by setting the ``firewall-driver``
   configuration option to 'openvswitch'.

   .. code-block:: none

      juju config neutron-openvswitch firewall-driver=openvswitch

7. Pause neutron-openvswitch and/or neutron-gateway units.

   If your deployments have two neutron-gateway units and four
   neutron-openvswitch units the sequence of commands would be:

   .. code-block:: none

      juju run-action neutron-gateway/0 pause
      juju run-action neutron-gateway/1 pause
      juju run-action neutron-openvswitch/0 pause
      juju run-action neutron-openvswitch/1 pause
      juju run-action neutron-openvswitch/2 pause
      juju run-action neutron-openvswitch/3 pause

8. Deploy the Neutron OVN plugin application

   .. code-block:: none

      juju deploy neutron-api-plugin-ovn \
         --series focal \
         --config dns-servers=="1.1.1.1 8.8.8.8"

      juju add-relation neutron-api-plugin-ovn:neutron-plugin \
         neutron-api:neutron-plugin-api-subordinate
      juju add-relation neutron-api-plugin-ovn:certificates \
         vault:certificates
      juju add-relation neutron-api-plugin-ovn:ovsdb-cms ovn-central:ovsdb-cms

   The values to use for the ``dns-servers`` configuration option can be
   found by looking at what is set for the ``dns-servers`` configuration
   option on the neutron-openvswitch and/or neutron-gateway applications.

   .. note::

      The plugin will not be activated until the neutron-api
      ``manage-neutron-plugin-legacy-mode`` configuration option is changed in
      step 9.

9. Adjust MTU on overlay networks (if required)

   Now that 24 hours have passed since we reduced the MTU on the instances
   running in the cloud as described in step 1, we can update the MTU setting
   for each individual Neutron network:

   .. code-block:: none

      juju run-action --wait neutron-api-plugin-ovn/0 migrate-mtu

10. Enable the Neutron OVN plugin

    .. code-block:: none

       juju config neutron-api manage-neutron-plugin-legacy-mode=false

    Wait for the deployment to settle.

11. Pause the Neutron API units

    .. code-block:: none

       juju run-action neutron-api/0 pause
       juju run-action neutron-api/1 pause
       juju run-action neutron-api/2 pause

    Wait for the deployment to settle.

12. Perform initial synchronization of the Neutron and OVN databases

    .. code-block:: none

       juju run-action --wait neutron-api-plugin-ovn/0 migrate-ovn-db

13. (Optional) Perform Neutron database surgery to update ``network_type`` of
    overlay networks to 'geneve'.

    At the time of this writing the Neutron OVN ML2 driver will assume that all
    chassis participating in a network are using the 'geneve' tunnel protocol
    and it will ignore the value of the `network_type` field in any
    non-physical network in the Neutron database. It will also ignore the
    `segmentation_id` field and let OVN assign the VNIs.

    The Neutron API currently does not support changing the type of a network,
    so when doing a migration the above described behaviour is actually a
    welcome one.

    However, after the migration is done and all the primary functions are
    working, i.e. packets are forwarded. The end user of the cloud will be left
    with the false impression of their existing 'gre' or 'vxlan' typed networks
    still being operational on said tunnel protocols, while in reality 'geneve'
    is used under the hood.

    The end user will also run into issues with modifying any existing networks
    with `openstack network set` throwing error messages about networks of type
    'gre' or 'vxlan' not being supported.

    After running this action said networks will have their `network_type`
    field changed to 'geneve' which will fix the above described problems.

    .. code-block:: none

       juju run-action --wait neutron-api-plugin-ovn/0 offline-neutron-morph-db

14. Resume the Neutron API units

    .. code-block:: none

       juju run-action neutron-api/0 resume
       juju run-action neutron-api/1 resume
       juju run-action neutron-api/2 resume

    Wait for the deployment to settle.

15. Migrate hypervisors and gateways

    The final step of the migration is to clean up after the Neutron agents
    on the hypervisors/gateways and enable the OVN services so that they can
    reprogram the local Open vSwitch.

    This can be done one gateway / hypervisor at a time or all at once to your
    discretion.

    .. note::

       During the migration instances running on a non-migrated hypervisor will
       not be able to reach instances on the migrated hypervisors.

    .. caution::

       When migrating a cloud with Neutron ML2+OVS+DVR+SNAT topology care should
       be taken to take into account on which hypervisors essential agents are
       running to minimize downtime for any instances on other hypervisors with
       dependencies on them.

    .. code-block:: none

       juju run-action --wait neutron-openvswitch/0 cleanup
       juju run-action --wait ovn-chassis/0 resume

       juju run-action --wait neutron-gateway/0 cleanup
       juju run-action --wait ovn-dedicated-chassis/0 resume

16. Post migration tasks

    Remove the now redundant Neutron ML2+OVS agents from hypervisors and
    any dedicated gateways as well as the neutron-gateway and
    neutron-openvswitch applications from the Juju model:

    .. code-block:: none

       juju run --application neutron-gateway '\
          apt remove -y neutron-dhcp-agent neutron-l3-agent \
          neutron-metadata-agent neutron-openvswitch-agent'

       juju remove-application neutron-gateway

       juju run --application neutron-openvswitch '\
          apt remove -y neutron-dhcp-agent neutron-l3-agent \
          neutron-metadata-agent neutron-openvswitch-agent'

       juju remove-application neutron-openvswitch

    Remove the now redundant Neutron ML2+OVS agents from the Neutron database:

    .. code-block:: none

       openstack network agent list
       openstack network agent delete ...

.. LINKS
.. _vault charm: https://charmhub.io/vault
