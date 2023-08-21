====================================================
LVM support in cinder charm: migration to cinder-lvm
====================================================

.. important::

   This page has been identified as being affected by the breaking changes
   introduced between versions 2.9.x and 3.x of the Juju client. Read
   support note :ref:`juju_29_3x_changes` before continuing.

As of the 21.10 release of OpenStack Charms, support for local (LVM) Cinder
storage in the `cinder`_ charm is deprecated. This functionality has been
de-coupled and is now managed by the `cinder-lvm`_ subordinate charm. This page
shows how to migrate from the cinder charm to the cinder-lvm charm.

.. warning::

   The migration will necessitate a short cloud maintenance window, enough time
   to deploy the subordinate charm onto the existing machine(s). Cloud
   operators will be unable to create new Cinder volumes during this window.

The LVM feature in the cinder charm is potentially in use if the
``block-device`` option is set to a value other than 'None'. Check this with
the following command:

.. code-block:: none

   juju config cinder block-device

Begin by disabling the LVM feature in the cinder charm:

.. code-block:: none

   juju config cinder block-device=None

Migrating LVM functionality amounts to setting the cinder-lvm charm
configuration options to the same values as those used for the identically
named options in the cinder charm:

* ``block-device``
* ``ephemeral-unmount``
* ``remove-missing``
* ``remove-missing-force``
* ``overwrite``
* ``volume-group``

Secondly, the ``volume-backend-name`` option specific to the cinder-lvm charm
needs to be set to 'cinder_lvm', the LVM backend driver.

All of this configuration can be stated most easily with a configuration file,
say ``cinder-lvm.yaml``:

.. code-block:: yaml

   cinder-lvm:
     block-device: sdb
     ephemeral-unmount: <null>
     remove-missing: false
     remove-missing-force: false
     overwrite: false
     volume-group: cinder-volumes
     volume-backend-name: cinder_lvm

Now deploy cinder-lvm while referencing the configuration file and then add a
relation to the cinder application:

.. code-block:: none

   juju deploy --config cinder-lvm.yaml cinder-lvm
   juju integrate cinder-lvm:storage-backend cinder:storage-backend

Verify that Cinder volumes can be created as usual and that existing VMs
utilising Cinder volumes have not been adversely affected.

.. LINKS
.. _cinder: https://charmhub.io/cinder
.. _cinder-lvm: https://charmhub.io/cinder-lvm
