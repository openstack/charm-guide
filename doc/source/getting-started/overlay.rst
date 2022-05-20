============================
Configure the overlay bundle
============================

The deployment will make use of the ``openstack-base`` bundle, which represents
the core of a Charmed OpenStack cloud. A separate overlay bundle is used to
tailor the configuration (possibly overriding setting in the bundle) so that
the deploy will work within a given environment.

.. note::

   Although it is possible to edit the bundle file itself, it is best practice
   to keep it pristine and to use an overlay instead.

The overlay file is :download:`overlay-focal-yoga-mymaas.yaml`. Download it and
save it in the ``~/tutorial`` directory.

Replace the variables in the file with the some of the values that were
collected in the :doc:`previous step <settings>`.

If constraints are not being used, replace ``$CONSTRAINTS`` with a null value
(nothing).

.. note::

   If a network space is configured within MAAS for the cloud's subnet, each
   charm will need to be informed of it via the ``bindings`` charm parameter.

Once you've edited and saved the file, proceed to the :doc:`Prepare Juju
<juju>` page.
