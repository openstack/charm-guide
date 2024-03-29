:orphan:

===============================
Bug triage and software reviews
===============================

The OpenStack Charms project operates two rotas for managing bug triage and
software reviews: one associated with triage of new bugs and the other related
to the review of software changes in Gerrit.

Community members are encouraged to sign up to both rotas. Please
:doc:`coordinate <contact>` with us!

Bug tags
--------

When triaging or filing bugs for `OpenStack Charms projects`_, please use the
following Launchpad tags:

* `openstack-upgrade`_ - Issues upgrading the charm payload (OpenStack
  version), such as ``Zed`` to ``2023.1``.
* `series-upgrade`_ - Issues upgrading from one series to the next, such as
  ``focal`` to ``jammy``.
* `charm-upgrade`_ - Issues upgrading the charm revision, such as ``ch:foo-100``
  to ``ch:foo-101`` (not a payload or series upgrade).
* `ceph-upgrade`_ - Issues upgrading the Ceph version (not charm upgrade).
* `scaleback`_ - Issues removing a unit, shrinking a cluster, replacing a unit.
* `cold-start`_ - Issues in recovering charm payload functionality after a
  power event such as a reboot or shutdown.
* `cross-model`_ - Issues with cross-model relations.
* `unstable-test`_ - Issues that result in :doc:`automated tests
  <software-contrib/testing>` producing false negatives or positives.

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
| gnuoy           | fnordahl   | freyes    | james-page | billy-olsen  |
+-----------------+------------+-----------+------------+--------------+
| aurelien-lourot | chris-icey | dmitriis  | coreycb    | alex-tinwood |
+-----------------+------------+-----------+------------+--------------+

Charm review
~~~~~~~~~~~~

For reviewing `all open changes`_ and `all reviewable items`_ across the
OpenStack Charms.

+----------+------------+--------------+-------------+-----------------+
| Monday   | Tuesday    | Wednesday    | Thursday    | Friday          |
+==========+============+==============+=============+=================+
| freyes   | james-page | fnordahl     | billy-olsen | chris-icey      |
+----------+------------+--------------+-------------+-----------------+
| dmitriis | coreycb    | alex-tinwood | gnuoy       | aurelien-lourot |
+----------+------------+--------------+-------------+-----------------+

Filtering in Gerrit
^^^^^^^^^^^^^^^^^^^

Sophisticated `filtering`_ is possible in Gerrit. For instance, to find open
changes to YAML files that have been in the queue for less than two months and
do not have a value of '-1' for the Workflow label: `open changes to YAML
files`_.

Abandonment policy
^^^^^^^^^^^^^^^^^^

Charm reviews which are either `blocked`_ (> 12 weeks without comment and
are blocked by a core reviewer with a -2 review) or `failing`_ (> 12 weeks
without comment and failed tests the last time the review was checked) will
be automatically marked as abandoned within Gerrit.

Abandoned reviews can always be restored in the event that work on a review
resumes - reviews that are blocked by a -2 review will also need to be
discussed with the core reviewer involved.

Any review marked as work in progress (Workflow -1) will be excluded from
this policy.

.. LINKS
.. _new bugs: https://bugs.launchpad.net/openstack-charms/+bugs?search=Search&field.status=New&orderby=-id&start=0
.. _all open changes: https://review.opendev.org/q/project:%22%255Eopenstack/charm.*%22+status:open
.. _all reviewable items: https://review.opendev.org/q/project:%22%255Eopenstack/charm.*%22+status:open+label:Verified%252B1+NOT+label:Verified-1+NOT+label:Code-Review-1
.. _filtering: https://review.opendev.org/Documentation/user-search.html
.. _open changes to YAML files: https://review.opendev.org/#/q/project:%22%255Eopenstack/charm-.*%22+status:open+file:%255E.*%255C.yaml+NOT+label:Workflow-1+NOT+age:2month
.. _OpenStack Charms projects: https://launchpad.net/openstack-charms
.. _charm-upgrade: https://bugs.launchpad.net/bugs/+bugs?field.tag=charm-upgrade
.. _series-upgrade: https://bugs.launchpad.net/bugs/+bugs?field.tag=series-upgrade
.. _openstack-upgrade: https://bugs.launchpad.net/bugs/+bugs?field.tag=openstack-upgrade
.. _ceph-upgrade: https://bugs.launchpad.net/bugs/+bugs?field.tag=ceph-upgrade
.. _scaleback: https://bugs.launchpad.net/bugs/+bugs?field.tag=scaleback
.. _cold-start: https://bugs.launchpad.net/bugs/+bugs?field.tag=cold-start
.. _cross-model: https://bugs.launchpad.net/bugs/+bugs?field.tag=cross-model
.. _unstable-test: https://bugs.launchpad.net/bugs/+bugs?field.tag=unstable-test
.. _failing: https://review.opendev.org/q/status:open+project:%255Eopenstack/charm-.*+age:12w+NOT+label:Verified%253E%253D1%252Cuosci-testing-bot%2540ubuntu.com+NOT+label:Workflow%253C%253D-1
.. _blocked: https://review.opendev.org/q/status:open+project:%255Eopenstack/charm-.*+age:12w+label:Code-Review%253C%253D-2+NOT+label:Workflow%253C%253D-1
