.. _rotas:

Rotas
=====

The OpenStack Charms development team operates two rotas; one associated
with triage of new, incoming bugs and the other related to the charm
review queue in Gerrit.

The development and user community are encouraged to sign up to both rotas.

Please coordinate activities in #openstack-charms on Freenode IRC.

Bug triage
++++++++++

Review `new bugs`_ across the OpenStack Charms.

+--------+------------+-----------+------------+--------------+
| Monday |  Tuesday   | Wednesday | Thursday   | Friday       |
+========+============+===========+============+==============+
| gnuoy  |  fnordahl  |  thedac   | james-page | billy-olsen  |
+--------+------------+-----------+------------+--------------+
|        | chris-icey |           | coreycb    | alex-tinwood |
+--------+------------+-----------+------------+--------------+

Charm review
++++++++++++

Review `open changes`_ across the OpenStack Charms.

+--------+------------+--------------+-------------+------------+
| Monday | Tuesday    | Wednesday    | Thursday    |  Friday    |
+========+============+==============+=============+============+
| thedac | james-page | fnordahl     | billy-olsen | chris-icey |
+--------+------------+--------------+-------------+------------+
|        | coreycb    | alex-tinwood | gnuoy       |            |
+--------+------------+--------------+-------------+------------+

Filtering in Gerrit
~~~~~~~~~~~~~~~~~~~

Sophisticated `filtering`_ is possible in Gerrit. For instance, to find open
changes to YAML files that have been sitting in the queue for less than two
months and do not have a value of '-1' for the Workflow label: `open changes to
YAML files`_.

.. LINKS
.. _`new bugs`: https://bugs.launchpad.net/openstack-charms/+bugs?search=Search&field.status=New&orderby=-id&start=0
.. _`open changes`: https://review.opendev.org/q/project:%22%255Eopenstack/charm.*%22+status:open
.. _`filtering`: https://review.opendev.org/Documentation/user-search.html
.. _`open changes to YAML files`: https://review.opendev.org/#/q/project:%22%255Eopenstack/charm-.*%22+status:open+file:%255E.*%255C.yaml+NOT+label:Workflow-1+NOT+age:2month
