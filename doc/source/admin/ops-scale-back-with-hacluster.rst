:orphan:

==================================================
Scale back an application with the hacluster charm
==================================================

Introduction
------------

This article shows how to scale back an application that is made highly
available by means of the subordinate hacluster charm. It implies the removal
of one or more of the principal application's units. This is easily done with
generic Juju commands and actions available to the hacluster charm.

.. note::

   Since the application being scaled back is already in HA mode the removal of
   one of its cluster members should not cause any immediate interruption of
   cloud services.

   Scaling back an application will also remove its associated hacluster unit.
   It is best practice to have at least three hacluster units per application
   at all times. An odd number is also recommended.

Procedure
---------

If the unit being removed is in a 'lost' state (as seen in :command:`juju
status`) please first see the `Notes`_ section.

List the application units
~~~~~~~~~~~~~~~~~~~~~~~~~~

Display the units, in this case for the vault application:

.. code-block:: none

   juju status vault

This article will be based on the following output:

.. code-block:: console

   Unit                     Workload  Agent  Machine  Public address  Ports     Message
   vault/0*                 active    idle   0/lxd/5  10.0.0.227      8200/tcp  Unit is ready (active: true, mlock: disabled)
     vault-hacluster/0*     active    idle            10.0.0.227                Unit is ready and clustered
     vault-mysql-router/0*  active    idle            10.0.0.227                Unit is ready
   vault/1                  active    idle   1/lxd/5  10.0.0.234      8200/tcp  Unit is ready (active: true, mlock: disabled)
     vault-hacluster/1      active    idle            10.0.0.234                Unit is ready and clustered
     vault-mysql-router/1   active    idle            10.0.0.234                Unit is ready
   vault/2                  active    idle   2/lxd/6  10.0.0.233      8200/tcp  Unit is ready (active: true, mlock: disabled)
     vault-hacluster/2      active    idle            10.0.0.233                Unit is ready and clustered
     vault-mysql-router/2   active    idle            10.0.0.233                Unit is ready

In the below example, unit ``vault/1`` will be removed.

Pause the subordinate hacluster unit
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Pause the hacluster unit that corresponds to the principle application unit
being removed. Here, unit ``vault-hacluster/1`` corresponds to unit
``vault/1``:

.. code-block:: none

   juju run-action --wait vault-hacluster/1 pause

.. caution::

   Unit numbers for a subordinate unit and its corresponding principal unit are
   not necessarily the same (e.g. it is possible to have ``vault-hacluster/2``
   correspond to ``vault/1``).

Remove the principal application unit
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Remove the principal application unit:

.. code-block:: none

   juju remove-unit vault/1

This will also remove the hacluster subordinate unit (and any other subordinate
units).

Update the ``cluster_count`` value
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Inform the hacluster charm about the new number of hacluster units, two here:

.. code-block:: none

   juju config vault-hacluster cluster_count=2

In this example a count of two (less than three) removes quorum functionality
and enables a two-node cluster. This is a sub-optimal state and is shown as an
example only.

Update Corosync
~~~~~~~~~~~~~~~

Remove Corosync nodes from its ring and update ``corosync.conf`` to reflect the
new number of nodes (``min_quorum`` is recalculated):

.. code-block:: none

   juju run-action --wait vault-hacluster/leader update-ring i-really-mean-it=true

Check the status of the Corosync cluster by querying a remaining hacluster
unit:

.. code-block:: none

   juju ssh vault-hacluster/leader sudo crm status

There should not be any node listed as OFFLINE.

.. note::

   With Juju client < 2.9 a subordinate leader unit must be referenced via its
   machine ID (e.g. 0/lxd/5) when using the :command:`juju ssh` command.

Verify cloud services
~~~~~~~~~~~~~~~~~~~~~

For this example, the final :command:`juju status vault` output is:

.. code-block:: console

   Unit                     Workload  Agent  Machine  Public address  Ports     Message
   vault/0*                 active    idle   0/lxd/5  10.0.0.227      8200/tcp  Unit is ready (active: true, mlock: disabled)
     vault-hacluster/0*     active    idle            10.0.0.227                Unit is ready and clustered
     vault-mysql-router/0*  active    idle            10.0.0.227                Unit is ready
   vault/2                  active    idle   2/lxd/6  10.0.0.233      8200/tcp  Unit is ready (active: true, mlock: disabled)
     vault-hacluster/2      active    idle            10.0.0.233                Unit is ready and clustered
     vault-mysql-router/2   active    idle            10.0.0.233                Unit is ready

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
