.. _support-notes:

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
   application in a second model. This scenario is documented in `Ceph RBD
   mirroring`_.
#. A ceph-radosgw application in one model, related to another ceph-radosgw
   application in a second model. This scenario is documented in `Ceph RADOS
   Gateway multisite replication`_.

Other scenarios may work but are currently not supported. Distinct development
and testing efforts are required in order to qualify each unique CMR scenario.

If you would like to help drive the development work towards implementing a
supported CMR scenario file a Launchpad bug describing your use case. To assist
in this effort kindly use the `cross-model`_ bug tag.

.. LINKS
.. _Cross model relations: https://juju.is/docs/cross-model-relations
.. _cross-model: https://bugs.launchpad.net/bugs/+bugs?field.tag=cross-model
.. _Ceph RBD Mirroring: https://docs.openstack.org/project-deploy-guide/charm-deployment-guide/latest/app-ceph-rbd-mirror.html
.. _Ceph RADOS Gateway Multisite replication: https://docs.openstack.org/project-deploy-guide/charm-deployment-guide/latest/app-rgw-multisite.html
