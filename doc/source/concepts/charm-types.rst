===========
Charm types
===========

There are two general types of OpenStack charms: one that does use channels and
one that does not (legacy).

Channels
--------

With the channels type, a channel is dedicated to a single OpenStack release
(release N-1 will be technically supported to assist with upgrades). This means
that a charm that works for a recent series-openstack combination will
generally not work on an older combination. Furthermore, there is a need to
switch to a different channel in order to upgrade to a new OpenStack version
- but not to a new series.

The :doc:`../project/charm-delivery` page explains how channel charms are
distributed to the end user.

Legacy
------

For the legacy charms, unless stated otherwise, each new revision of a charm
includes all the functionality of the previous revision. This means that a
charm that works for a recent series-openstack combination will also work on an
older combination.

The development of legacy charms has stopped at the 21.10 release of OpenStack
Charms (and at the 21.06 release of Trilio Charms). The last supported
series-openstack combination is ``focal-xena``.

