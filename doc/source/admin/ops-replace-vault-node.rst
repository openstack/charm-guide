:orphan:

==========================
Replace Vault cluster node
==========================

Introduction
------------

This article shows how to replace a Vault node in a cluster made highly
available by means of the subordinate hacluster charm. It implies the removal
and then the addition of a vault unit. This is done with generic Juju commands
and actions available to the hacluster charm.

.. important::

   This procedure will not result in cloud downtime providing that there is at
   least one functional Vault node present at all times.

.. warning::

   This procedure will involve a sealed Vault instance. Please ensure that the
   requisite number of unseal keys are available before continuing.

Procedure
---------

If the unit being removed is in a 'lost' state (as seen in :command:`juju
status`) please first see the `Notes`_ section.

List the application units
~~~~~~~~~~~~~~~~~~~~~~~~~~

Display the units, in this case for the vault application:

.. code-block:: none

   juju status vault

This article will be based on the following (partial) output:

.. code-block:: console

   Unit                     Workload  Agent  Machine  Public address  Ports     Message
   vault/0*                 active    idle   1/lxd/4  10.246.114.76   8200/tcp  Unit is ready (active: false, mlock: disabled)
     vault-hacluster/1      active    idle            10.246.114.76             Unit is ready and clustered
     vault-mysql-router/0*  active    idle            10.246.114.76             Unit is ready
   vault/3                  active    idle   0/lxd/8  10.246.114.83   8200/tcp  Unit is ready (active: true, mlock: disabled)
     vault-hacluster/2      active    idle            10.246.114.83             Unit is ready and clustered
     vault-mysql-router/25  active    idle            10.246.114.83             Unit is ready
   vault/4                  active    idle   2/lxd/9  10.246.114.84   8200/tcp  Unit is ready (active: false, mlock: disabled)
     vault-hacluster/0*     active    idle            10.246.114.84             Unit is ready and clustered
     vault-mysql-router/24  active    idle            10.246.114.84             Unit is ready

In this example, unit ``vault/3`` will be removed.

Pause the subordinate hacluster unit
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Pause the hacluster unit that corresponds to the principal application unit
being removed. Here, unit ``vault-hacluster/2`` corresponds to unit
``vault/3``:

.. code-block:: none

   juju run-action --wait vault-hacluster/2 pause

Remove the principal application unit
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Remove the principal application unit:

.. code-block:: none

   juju remove-unit vault/3

This will also remove the hacluster subordinate unit (and any other subordinate
units).

Add a principal application unit
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Scale out the existing vault application and place the new (containerised) unit
on the same host that the removed unit was on (machine 0):

.. code-block:: none

   juju add-unit --to lxd:0 vault

.. caution::

   If network spaces are in use the above command will not succeed. See Juju
   issue `LP #1969523`_ for a workaround.

The new :command:`juju status` output now contains:

.. code-block:: console

   Unit                     Workload  Agent  Machine  Public address  Ports     Message
   vault/0*                 active    idle   1/lxd/4  10.246.114.76   8200/tcp  Unit is ready (active: false, mlock: disabled)
     vault-hacluster/1      active    idle            10.246.114.76             Unit is ready and clustered
     vault-mysql-router/0*  active    idle            10.246.114.76             Unit is ready
   vault/4                  active    idle   2/lxd/9  10.246.114.84   8200/tcp  Unit is ready (active: true, mlock: disabled)
     vault-hacluster/0*     active    idle            10.246.114.84             Unit is ready and clustered
     vault-mysql-router/24  active    idle            10.246.114.84             Unit is ready
   vault/6                  blocked   idle   0/lxd/9  10.246.114.83   8200/tcp  Unit is sealed
     vault-hacluster/28     active    idle            10.246.114.83             Unit is ready and clustered
     vault-mysql-router/40  active    idle            10.246.114.83             Unit is ready

Notice that the new vault unit (``vault/6``) is sealed.

Unseal the new Vault instance
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Here we will assume that the original Vault deploy was initialised with a
requirement of three unseal keys.

Set an environment variable based on the address of the newly-introduced unit,
and unseal the instance:

.. code-block:: none

   export VAULT_ADDR="http://10.246.114.83:8200"

   vault operator unseal
   vault operator unseal
   vault operator unseal

For more information on unsealing Vault see cloud operation :doc:`Unseal Vault
<ops-unseal-vault>`.

Verify cloud services
~~~~~~~~~~~~~~~~~~~~~

The final :command:`juju status vault` (partial) output is:

.. code-block:: console

   Unit                     Workload  Agent  Machine  Public address  Ports     Message
   vault/0*                 active    idle   1/lxd/4  10.246.114.76   8200/tcp  Unit is ready (active: false, mlock: disabled)
     vault-hacluster/1      active    idle            10.246.114.76             Unit is ready and clustered
     vault-mysql-router/0*  active    idle            10.246.114.76             Unit is ready
   vault/4                  active    idle   2/lxd/9  10.246.114.84   8200/tcp  Unit is ready (active: true, mlock: disabled)
     vault-hacluster/0*     active    idle            10.246.114.84             Unit is ready and clustered
     vault-mysql-router/24  active    idle            10.246.114.84             Unit is ready
   vault/6                  active    idle   0/lxd/9  10.246.114.83   8200/tcp  Unit is ready (active: false, mlock: disabled)
     vault-hacluster/28     active    idle            10.246.114.83             Unit is ready and clustered
     vault-mysql-router/40  active    idle            10.246.114.83             Unit is ready

Ensure that all cloud services are working as expected.

Notes
-----

Pre-removal, in the case where the principal application unit has transitioned
to a 'lost' state (e.g. dropped off the network due to a hardware failure),

#. the first step (pause the hacluster unit) can be skipped
#. the second step (remove the principal unit) can be replaced by:

   .. code-block:: none

      juju remove-machine N --force

   N is the Juju machine ID (see the :command:`juju status` command) where the
   unit to be removed is running.

   .. warning::

      Removing the machine by force will naturally remove any other units that
      may be present, including those from an entirely different application.

.. LINKS
.. _LP #1969523: https://bugs.launchpad.net/juju/+bug/1969523
