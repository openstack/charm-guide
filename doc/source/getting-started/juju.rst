============
Prepare Juju
============

To use MAAS as a backing cloud we need to add MAAS to Juju, give MAAS
credentials to Juju, and create a Juju controller. The corresponding Juju
commands are:

.. code-block:: none

   juju add-cloud
   juju add-credential
   juju bootstrap

Please see the Juju documentation for guidance: `How to use MAAS with Juju`_.

Assuming the controller is called 'maas-controller', create a model called
'openstack' and give it the desired default series:

.. code-block:: none

   juju add-model -c maas-controller --config default-series=focal openstack

Advance to the :doc:`Deploy OpenStack <deploy>` page.

.. LINKS
.. _How to use MAAS with Juju: https://jaas.ai/docs/maas-cloud
