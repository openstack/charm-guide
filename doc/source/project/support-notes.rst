=============
Support notes
=============

This page describes supportability aspects within the OpenStack Charms project.

Cross model relations
---------------------

Juju relations that are added between applications that reside in separate Juju
models are called `Cross model relations`_ (CMR). For a Charmed OpenStack
deployment, CMR support must also be present in terms of the manner in which
the charms are combined. The OpenStack Charms do not generically support CMR.

.. important::

   It is recommended to exercise each new CMR scenario in a lab environment
   before committing to that topology in production.

The current list of supported CMR scenarios with regard to OpenStack Charms is
as follows:

#. The keystone application in one model, related to the vault application in a
   second model.
#. The ceph-mon application in one model, related to the ceph-rbd-mirror
   application in a second model. This scenario is documented in
   :doc:`../admin/storage/ceph-rbd-mirror`.
#. A ceph-radosgw application in one model, related to another ceph-radosgw
   application in a second model. This scenario is documented in
   :doc:`../admin/storage/ceph-rgw-multisite`.

Other scenarios may work but are currently not supported. Distinct development
and testing efforts are required in order to qualify each unique CMR scenario.

If you would like to help drive the development work towards implementing a
supported CMR scenario file a Launchpad bug describing your use case. To assist
in this effort kindly use the `cross-model`_ bug tag.

.. _juju_29_3x_changes:

Breaking changes between Juju 2.9.x and 3.x
-------------------------------------------

As this guide is written for Juju 3.x, users of 2.9.x will encounter noteworthy
changes when following the instructions. This section explains those changes.

Breaking changes have been introduced in the Juju client between versions 2.9.x
and 3.x. These are caused by the renaming and re-purposing of several commands
- functionality and command options remain unchanged.

In the context of this guide, the pertinent changes are shown here:

+---------------------------+----------------------------+
| 2.9.x                     | 3.x                        |
+===========================+============================+
| :command:`add-relation`   | :command:`integrate`       |
+---------------------------+----------------------------+
| :command:`run`            | :command:`exec`            |
+---------------------------+----------------------------+
| :command:`run-action`     | :command:`run`             |
+---------------------------+----------------------------+

See the `Juju 3.0 release notes`_ for the comprehensive list of changes.

The response is to therefore substitute the documented command with the
equivalent 2.9.x command. For example:

Juju 3.x,

.. code-block:: none

   juju integrate keystone-hacluster:ha keystone:ha

Juju 2.9.x,

.. code-block:: none

   juju add-relation keystone-hacluster:ha keystone:ha

.. note::

   Pages in this guide impacted by these name changes have had an admonishment
   added to them that points back to this present page.

.. LINKS
.. _Cross model relations: https://juju.is/docs/juju/cross-model-integration
.. _cross-model: https://bugs.launchpad.net/bugs/+bugs?field.tag=cross-model
.. _Juju 3.0 release notes: https://juju.is/docs/juju/roadmap#heading--juju-3-0-0---22-oct-2022
