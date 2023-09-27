================
Deploy OpenStack
================

.. important::

   This page has been identified as being affected by the breaking changes
   introduced between versions 2.9.x and 3.x of the Juju client. Read
   support note :ref:`juju_29_3x_changes` before continuing.

Ensure that the current context is the previously-created controller and model:

.. code-block:: none

   juju switch maas-controller:openstack

Download bundle file :download:`bundle-jammy-2023.1.yaml` and save it in the
``~/tutorial`` directory.

Enter the tutorial directory and deploy OpenStack by referring to the bundle
and the overlay:

.. code-block:: none

   cd ~/tutorial
   juju deploy ./bundle-jammy-2023.1.yaml --overlay ./overlay-mymaas.yaml

This stage of the procedure can take between 30 and 90 minutes to complete,
depending on how the MAAS nodes are resourced. Use the :command:`juju status`
command to monitor progress.

Vault requires manual intervention in order to become functional. Complete the
three post-deployment steps (initialisation, unsealing, authorisation)
described in the `vault charm README`_ when the vault application shows a
workload status of: ``Vault needs to be initialized`` .

A CA certificate will then need to be supplied to Vault so it can issue TLS
certificates to the various cloud services. The easiest approach is to have
Vault generate the CA certificate:

.. code-block:: none

   juju run vault/leader generate-root-ca

See the :doc:`../admin/security/tls` page for further guidance.

Allow the model to settle to an error-free state (see this :ref:`example status
output <juju_status>`).

When you're ready, go to the :doc:`openstack` page.

.. LINKS
.. _Vault charm README: https://opendev.org/openstack/charm-vault/src/branch/master/src/README.md#post-deployment-tasks
