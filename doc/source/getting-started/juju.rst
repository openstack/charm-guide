============
Prepare Juju
============

To use MAAS as a backing cloud we need to:

1. add MAAS to Juju
1. give MAAS credentials to Juju
1. create a Juju controller

The corresponding Juju commands are:

.. code-block:: none

   juju add-cloud
   juju add-credential
   juju bootstrap

Please see the Juju documentation for guidance: `How to use MAAS with Juju`_.
The above commands only give the general idea.

Assuming the controller is called 'maas-controller', create a model called
'openstack' and give it the desired default series:

.. code-block:: none

   juju add-model -c maas-controller --config default-series=focal openstack

Advance to the :doc:`Deploy OpenStack <deploy>` page.

.. LINKS
.. _How to use MAAS with Juju: https://juju.is/docs/olm/maas
