================
Deploy OpenStack
================

The actual bundle is found in the ``openstack-bundles`` repository maintained
by the OpenStack charmers team. Clone it and change to the bundle's directory:

.. code-block:: none

   git clone https://github.com/openstack-charmers/openstack-bundles ~/tutorial
   cd ~/tutorial/openstack-bundles/stable/openstack-base

The bundle is located in file ``bundle.yaml``.

Ensure that the current context is the previously-created controller and model:

.. code-block:: none

   juju switch maas-controller:openstack

Deploy the cloud like this:

.. code-block:: none

   juju deploy ./bundle.yaml --overlay ~/tutorial/overlay-openstack-base-focal-mymaas.yaml

This stage of the procedure can take between 30 and 90 minutes to complete,
depending on how the MAAS nodes are resourced. Use the :command:`juju status`
command to monitor progress.

Vault requires manual intervention in order to become functional. Complete the
steps described in the `vault charm README`_ when the vault application shows a
workload status of ``Vault needs to be initialized``. Once that's done, allow
the model to settle to an error-free state (see this :ref:`example status
output <getting_started_juju_status>`).

When you're ready, go to the :doc:`Configure OpenStack <openstack>` page.

.. LINKS
.. _Vault charm README: https://opendev.org/openstack/charm-vault/src/branch/master/src/README.md#post-deployment-tasks