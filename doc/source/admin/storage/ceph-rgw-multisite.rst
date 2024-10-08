=========================================
Ceph RADOS Gateway multi-site replication
=========================================

Ceph RADOS Gateway (RGW) native replication between ceph-radosgw applications
is supported both within a single Juju model and between separate models. When
using multiple models, Juju cross-model relations are leveraged.

Typically, each ceph-radosgw deployment is associated with a distinct Ceph
cluster in different physical locations, in which case separate models will be
used.

Multi-site can either be set up at the initial deploy time of both clusters or
added on to a pre-existing cluster.

In the event that the primary site has an outage, it can be failed over to the
secondary site.

Finally, scaling down multi-site to single site is supported.

Deployment and configuration details are available in the `Charmed Ceph
documentation`_.

.. LINKS
.. _Charmed Ceph documentation: https://ubuntu.com/ceph/docs/setting-up-multi-site-replication
