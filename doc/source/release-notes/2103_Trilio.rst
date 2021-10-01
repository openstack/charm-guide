.. _release_notes_trilio_21.03:

============
21.03 Trilio
============

Summary
-------

The 21.03 release of the Trilio charms adds support for TrilioVault 4.1.

.. important::

   The Trilio charms are no longer released with the same cadence as the other
   OpenStack charms. Instead, they are released shortly after releases of the
   Trilio code.

Supported combinations
----------------------

The below table shows the support status for versions of TrilioVault for each
version of Ubuntu and OpenStack.

.. list-table:: **Supported combinations**
   :header-rows: 1
   :widths: 12 12 20 20

   * - Ubuntu
     - OpenStack
     - TrilioVault 4.0
     - TrilioVault 4.1

   * - Bionic
     - Queens
     - ✔
     - ✔

   * - Bionic
     - Stein
     - ✔
     - ✔

   * - Bionic
     - Train
     - ✔
     - ✔

   * - Bionic
     - Ussuri
     - X
     - ✔

   * - Focal
     - Ussuri
     - X
     - ✔

Upgrading to 21.03 Trilio charms
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The Trilio charms are upgraded in the usual way:

.. code-block:: none

   juju upgrade-charm trilio-data-move
   juju upgrade-charm trilio-dm-api
   juju upgrade-charm trilio-horizon-plugin
   juju upgrade-charm trilio-wlm

Once all the charms are upgraded, the trilio-data-mover charm will go into a
blocked state as it now requires a relation with the cloud database. To add
the relation:

.. code-block:: none

   juju add-relation trilio-data-mover:shared-db percona-cluster:shared-db

Upgrading to TrilioVault 4.1
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The deployed TrilioVault version is controlled by the
``triliovault-pkg-source`` charm configuration option. To upgrade, update this
option's value for each of the deployed Trilio applications:

.. code-block:: none

   juju config trilio-wlm triliovault-pkg-source="deb [trusted=yes] https://apt.fury.io/triliodata-4-1/ /"
   juju config trilio-dm-api triliovault-pkg-source="deb [trusted=yes] https://apt.fury.io/triliodata-4-1/ /"
   juju config trilio-data-mover triliovault-pkg-source="deb [trusted=yes] https://apt.fury.io/triliodata-4-1/ /"
   juju config trilio-horizon-plugin triliovault-pkg-source="deb [trusted=yes] https://apt.fury.io/triliodata-4-1/ /"
