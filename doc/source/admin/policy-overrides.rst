=============================
Working with policy overrides
=============================

This page shows how to work with policy overrides. Specifically, it shows the
steps needed for enabling, updating, and disabling them. A tutorial on enabling
a policy override is also linked to.

.. important::

   Become familiar with the concepts behind policy overrides prior to
   attempting to use them. Consult therefore the
   :doc:`../concepts/policy-overrides` page before going forward.

Enable an override
------------------

A policy override for a single OpenStack service is enabled in four steps:

#. Insert the policy statements into an override file (or files).

   This creates an override file. Its contents is dependent upon the desired
   policy for the given service.

#. Compress the override file(s) to get the resource file:

   .. code-block:: none

      zip <resource-file.zip> <override-file.yaml> [<override-file.yaml> ...]

#. Attach the resource file to the application. The resource name used is
   ``policyd-override``:

   .. code-block:: none

      juju attach-resource <charm-name> policyd-override=<resource-file.zip>

#. Enable the override via the ``use-policyd-override`` charm option:

   .. code-block:: none

      juju config <charm-name> use-policyd-override=true

See tutorial :doc:`ops-show-extended-server-attributes` for a practical example
of enabling a policy override.

Resource file requirements
~~~~~~~~~~~~~~~~~~~~~~~~~~

The requirements for the resource file are:

* It must be properly ZIP formatted. A ``pkunzip`` program must be able to open
  and test the enclosed files.
* Enclosed override files must be properly YAML formatted and have an extension
  of ``.yaml``, or ``.yml``.
* Enclosed override files must **not** contain rule targets/keys that have been
  blacklisted by the charm. These will be documented in the charm's README.
* Enclosed override files must have unique filenames. Any directories in the
  file are "flattened" such that all override files appear as a simple list.
  Each of these filenames also get lower-cased.

Update an override
------------------

To update (or fix) an override attach a new resource file. Changes are applied
immediately; there is no need to disable ('false') and re-enable ('true').

.. note::

   The override that gets applied are always associated with the most recently
   attached resource file.

The last revision time of the resource can be viewed with the :command:`juju
list-resources` command. Sample output is:

.. code-block:: console

   Resource          Revision
   policyd-override  2020-03-12T19:53

Disable an override
-------------------

Overrides are disabled by setting option ``use-policyd-override`` back to its
default value of 'false':

.. code-block:: none

   juju config <charm-name> use-policyd-override=false

You do not need to remove the resource file. Indeed, there is no ability in
Juju to do so.

.. note::

   A charm that supports policy overrides will always have the
   'policyd-override' resource present.
