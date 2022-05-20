================
Deploy OpenStack
================

Ensure that the current context is the previously-created controller and model:

.. code-block:: none

   juju switch maas-controller:openstack

The bundle file is :download:`bundle-focal-yoga.yaml`. Download it and
save it in the ``~/tutorial`` directory.

Enter the tutorial directory and deploy OpenStack by referring to the bundle
and the overlay:

.. code-block:: none

   cd ~/tutorial
   juju deploy ./bundle-focal-yoga.yaml --overlay ./overlay-focal-yoga-mymaas.yaml

This stage of the procedure can take between 30 and 90 minutes to complete,
depending on how the MAAS nodes are resourced. Use the :command:`juju status`
command to monitor progress.

Vault requires manual intervention in order to become functional. Complete the
steps described in the `vault charm README`_ when the vault application shows a
workload status of ``Vault needs to be initialized``. Once that's done, allow
the model to settle to an error-free state (see this :ref:`example status
output <juju_status>`).

When you're ready, go to the :doc:`Configure OpenStack <openstack>` page.

.. LINKS
.. _Vault charm README: https://opendev.org/openstack/charm-vault/src/branch/master/src/README.md#post-deployment-tasks
