============
Prepare Juju
============

To use MAAS as a backing cloud we need to:

#. add MAAS to Juju
#. give MAAS credentials to Juju
#. create a Juju controller

The corresponding Juju commands are:

.. code-block:: none

   juju add-cloud
   juju add-credential
   juju bootstrap

The above will result in one MAAS node being provisioned.

Please see the Juju documentation for guidance: `How to use MAAS with Juju`_.
The above commands only give the general idea.

Assuming that the controller is called 'maas-controller', create a model called
'openstack' and give it the desired default series:

.. code-block:: none

   juju add-model -c maas-controller --config default-series=jammy openstack

Advance to the :doc:`deploy` page.

.. LINKS
.. _How to use MAAS with Juju: https://juju.is/docs/olm/maas
