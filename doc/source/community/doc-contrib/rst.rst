================
reStructuredText
================

This page provides guidance when working with the RST format.

General formatting
------------------

Italics - use single asterisks:

.. code-block:: none

   *this is in italics*

Bold - use double asterisks:

.. code-block:: none

   **this is in bold**

Monospace - use double backticks:

.. code-block:: none

   the ``--force`` option may help

Section headers
---------------

There are five section headers:

.. code-block:: none

   =======================
   H1 (double equal signs)
   =======================

   H2 (dashes)
   -----------

   H3 (tildes)
   ~~~~~~~~~~~

   H4 (carets)
   ^^^^^^^^^^^

   H5 (dots)
   .........

Inline commands
---------------

For commands or utilities that are mentioned in a sentence use the
``:command:`` directive:

.. code-block:: none

   You can type the :command:`juju status` command to get an overview of the
   model.

   The :command:`openstack` client is the preferred tool.

Linking to an external site
---------------------------

.. code-block:: none

   see the `Juju documentation`_ for more details ...

   .
   .

   The issue is tracked in bug `LP #1846279`_ ...

   .
   .
   <bottom of page>

   .. LINKS
   .. _Juju documentation: https://juju.is/docs

   .. BUGS
   .. _LP #1846279: https://bugs.launchpad.net/postgresql-charm/+bug/1846279

Linking to a page in the doc set
--------------------------------

Assuming that the destination document is ``install-maas.rst`` then in the
source document:

.. code-block:: none

   In the :doc:`previous section <install-maas>`

The linking is relative. If the destination document was in the parent
directory:

.. code-block:: none

   In the :doc:`previous section <../install-maas>`

Linking to a location within the current page
---------------------------------------------

Section headers are valid targets by default (implicit links).

.. code-block:: none

   Deploy OpenStack
   ~~~~~~~~~~~~~~~~

   .
   .

   In the `Deploy OpenStack`_ step above

First create a target in order to link to a non-header. Use one of three
methods:

.. code-block:: none

   In the example_ below

   or in `example #5`_

   or in the :ref:`Crisis situation <example_crisis>` example

   .
   .

   .. _example:

   .
   .

   .. _example #5:

   .
   .

   .. _example_crisis:

Linking to a location within a page in the doc set
--------------------------------------------------

In the source document:

.. code-block:: none

   during the :ref:`Install MAAS <install_maas>` step on the previous page

In the destination document insert the target code (typically above a section
header):

.. code-block:: none

   .. _install_maas:

   Install MAAS
   ------------

Admonishments
-------------

Admonishment types and their purpose:

+---------------+-----------------------------------------------+
| Type          | Purpose                                       |
+===============+===============================================+
| ``tip``       | to provide auxiliary information              |
+---------------+-----------------------------------------------+
| ``note``      | to inform                                     |
+---------------+-----------------------------------------------+
| ``important`` | to accentuate                                 |
+---------------+-----------------------------------------------+
| ``caution``   | to draw special attention to                  |
+---------------+-----------------------------------------------+
| ``warning``   | to warn about potential breakage or data loss |
+---------------+-----------------------------------------------+

Syntax:

.. code-block:: none

   .. <type>::

      text goes here. text goes here. text goes here. text goes here. text goes
      maintain the alignment.

The text is left-aligned with the admonishment type.

Example:

.. code-block:: none

   .. note::

      This is a note.

Code blocks
-----------

Syntax for code blocks:

.. code-block:: none

   .. code-block:: <type>

      something goes here

The block is left-aligned with 'code-block'.

Code block types:

+--------------------------------+----------------------------+
| Type                           | Purpose                    |
+================================+============================+
| ``none``                       | console input              |
+--------------------------------+----------------------------+
| ``console``                    | console output             |
+--------------------------------+----------------------------+
| ``python``, ``bash``, ``yaml`` | code snippets/scripts      |
+--------------------------------+----------------------------+
| ``ini``                        | miscellaneous file content |
+--------------------------------+----------------------------+

Console input
~~~~~~~~~~~~~

.. code-block:: none

   The following command shows the relations:

   .. code-block:: none

      juju status --relations

Console output
~~~~~~~~~~~~~~

.. code-block:: none

   Sample output of the last command is:

   .. code-block:: console

      Name            Version      Rev    Tracking        Publisher    Notes
      charm           2.8.2        609    latest/stable   canonical✓   classic
      charmcraft      1.4.0        761    latest/stable   canonical✓   classic

Code snippet
~~~~~~~~~~~~

.. code-block:: none

   This bit of Python will do the trick:

   .. code-block:: python

      def anagram(first, second):
       return Counter(first) == Counter(second)

Do not use the ``bash`` type for simple command invocations.

Miscellaneous file contents
~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: none

   The contents of file ``/etc/ec2_version`` is:

   .. code-block:: ini

      Ubuntu 20.04.1 LTS (Focal)

Lists
-----

Add a blank line between each item if any list items are multi-lined.

Unordered list
~~~~~~~~~~~~~~

.. code-block:: none

   * First item. Align any word-wrapped lines
     like this.

   * Second item

Nested unordered list
~~~~~~~~~~~~~~~~~~~~~

For nested lists, indent items so they align with the parent text:

.. code-block:: none

   * First item

     * Nested item
     * Nested item

   * First item

     * Nested item
     * Nested item

Ordered list
~~~~~~~~~~~~

.. code-block:: none

   #. First item
   #. Second item

Nested ordered list
~~~~~~~~~~~~~~~~~~~

For nested lists, indent items so they align with the parent text:

.. code-block:: none

   #. First item

      #. Nested item
      #. Nested item

   #. First item

      #. Nested item
      #. Nested item

Definitions
-----------

To define a term, indent its definition by two spaces:

.. code-block:: none

   Charm upgrade
     An upgrade of the charm software which is used to deploy and manage
     OpenStack. This includes charms that manage applications which are not
     technically part of the OpenStack project.

Images and figures
------------------

To insert an image or a figure:

.. code-block:: none

   .. image:: <relative/path/to/image.png>
      :<property>
      :<property>

See `RST documentation on images and figures`_ for details.

.. LINKS
.. _RST documentation on images and figures: https://docutils.sourceforge.io/docs/ref/rst/directives.html#images
