.. _rotas:

=====
Rotas
=====

The OpenStack Charms development team operates two rotas: one associated with
triage of new, incoming bugs and the other related to the charm review queue in
Gerrit.

The development and user community are encouraged to sign up to both rotas.

Please coordinate activities in #openstack-charms on Freenode IRC.

Bug tags
--------

When triaging or filing bugs for OpenStack Charms `projects`_, please use the
following Launchpad tags to classify relevant bugs:

* `openstack-upgrade`_ - Issues upgrading the charm payload (OpenStack
  version), such as Train to Ussuri.
* `charm-upgrade`_ - Issues upgrading the charm revision, such as cs:foo-100
  to cs:foo-101 (not a payload or OpenStack version upgrade, not a series
  upgrade).
* `series-upgrade`_ - Issues upgrading from one series to the next, ie. Bionic
  to Focal.
* `ceph-upgrade`_ - Issues upgrading the Ceph version (not charm upgrade).
* `scaleback`_ - Issues removing a unit, shrinking a cluster, replacing a unit.
* `cold-start`_ - Issues in recovering the charm payload functionality after a
  power event such as a reboot or shutdown.

Rota schedules
--------------

People are assigned to the rotas using an alternating daily schedule. The names
in the tables are Launchpad IDs.

Bug triage
~~~~~~~~~~

For triaging `new bugs`_ across the OpenStack Charms.

+-----------------+------------+-----------+------------+--------------+
| Monday          | Tuesday    | Wednesday | Thursday   | Friday       |
+=================+============+===========+============+==============+
| gnuoy           | fnordahl   | thedac    | james-page | billy-olsen  |
+-----------------+------------+-----------+------------+--------------+
| aurelien-lourot | chris-icey | dmitriis  | coreycb    | alex-tinwood |
+-----------------+------------+-----------+------------+--------------+

Charm review
~~~~~~~~~~~~

For reviewing `open changes`_ across the OpenStack Charms.

+----------+------------+--------------+-------------+-----------------+
| Monday   | Tuesday    | Wednesday    | Thursday    | Friday          |
+==========+============+==============+=============+=================+
| thedac   | james-page | fnordahl     | billy-olsen | chris-icey      |
+----------+------------+--------------+-------------+-----------------+
| dmitriis | coreycb    | alex-tinwood | gnuoy       | aurelien-lourot |
+----------+------------+--------------+-------------+-----------------+

Filtering in Gerrit
^^^^^^^^^^^^^^^^^^^

Sophisticated `filtering`_ is possible in Gerrit. For instance, to find open
changes to YAML files that have been in the queue for less than two months and
do not have a value of '-1' for the Workflow label: `open changes to YAML
files`_.

.. LINKS
.. _`new bugs`: https://bugs.launchpad.net/openstack-charms/+bugs?search=Search&field.status=New&orderby=-id&start=0
.. _`open changes`: https://review.opendev.org/q/project:%22%255Eopenstack/charm.*%22+status:open
.. _`filtering`: https://review.opendev.org/Documentation/user-search.html
.. _`open changes to YAML files`: https://review.opendev.org/#/q/project:%22%255Eopenstack/charm-.*%22+status:open+file:%255E.*%255C.yaml+NOT+label:Workflow-1+NOT+age:2month
.. _`projects`: https://launchpad.net/openstack-charms
.. _`charm-upgrade`: https://bugs.launchpad.net/bugs/+bugs?field.tag=charm-upgrade
.. _`series-upgrade`: https://bugs.launchpad.net/bugs/+bugs?field.tag=series-upgrade
.. _`openstack-upgrade`: https://bugs.launchpad.net/bugs/+bugs?field.tag=openstack-upgrade
.. _`ceph-upgrade`: https://bugs.launchpad.net/bugs/+bugs?field.tag=ceph-upgrade
.. _`scaleback`: https://bugs.launchpad.net/bugs/+bugs?field.tag=scaleback
.. _`cold-start`: https://bugs.launchpad.net/bugs/+bugs?field.tag=cold-start
