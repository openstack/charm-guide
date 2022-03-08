===================
Writing style guide
===================

Overview
--------

This guide describes the writing style that is applied to the OpenStack Charms
project documentation:

* `OpenStack Charm Guide`_
* `OpenStack Charms Deployment Guide`_
* `OpenStack charms`_ (README files)

Both the above guides are published using the `Sphinx`_ documentation
generator. As such, enhanced `reStructuredText`_ (RST) formatting is used.

The charm README files are formatted in Markdown (MD).

.. REPLACE THE ABOVE LINE WITH THIS ONCE THE CITED DOCUMENT IS MERGED
   The charm README files are formatted in Markdown. The :doc:`Charm README
   template <charm-readme-template>` provides guidance on how to produce a
   README file.

Other resources
~~~~~~~~~~~~~~~

The below resources contain a wealth of information that can also be of use:

* the `Canonical documentation style guide`_
* the `OpenStack documentation contributor guide`_

In general, the OpenStack Charms project abides by these guides. However, in
cases of disagreement or ambiguity the current document takes precedence over
them.

General guidelines
------------------

* Juju application names are not formatted:

  * "the nova-compute application"
  * "the nova-compute charm"
  * "the nova-compute unit"

* Cited bugs are expressed as hyperlinks and, depending on Launchpad or GitHub
  bug trackers, are of the form:

  * "see bug `LP #123456`_"
  * "tracked in bug `GH #173`_"

.. _LP #123456: https://pad.lv/123456
.. _GH #173: https://github.com/juju-solutions/layer-basic/issues/173

* Use monospace format for:

  * unit names: "the ``nova-compute/3`` unit"
  * action names: "the ``list-disks`` action"
  * references to utilities/programs: "like ``haproxy`` or ``curl``"
  * paths: "in the ``/etc/glance`` directory"
  * command options: "the ``--force`` command option"
  * charm configuration options: "the ``data-port`` option"

* Use a maximum line length of 79 characters.

* Use single quotes for values: "the option's value is 'br-ex:eth1'".

* Use bold and italics very sparingly.

* Use single spaces between sentences.

* Use hyperlink labels at the bottom of a page.

Admonishments
-------------

An admonishment should contain drop-in, self-sufficient text. That is, it
should not depend on the main-body (surrounding) text to be intelligible. In
this way, if the admonishment becomes no longer applicable, removing it will
not adversely affect the logic of the surrounding text.

Capitalisation
--------------

Capitalisation should be minimised:

* Use a capital letter for just the first word of all section headers
* Use capital letters for any proper nouns or acronyms, as usual
* OpenStack services such as Nova
* Project names like HAProxy or OpenStack

Transient information
---------------------

Be mindful of including information that is expected to become out of date,
such as citing bugs or listing things that will surely change. It might be
better to simply omit some information. For instance, do not start a list of
versions thinking that it will be maintained by someone. To avoid:

.. code-block:: none

   Firefly is available in Trusty, Hammer is in Trusty-Juno (end of life),
   Trusty-Kilo, Trusty-Liberty, and Jewel is available in Trusty-Mitaka.

The use of temporal expressions such as "not yet", "currently", and "at the
time of writing", or closely related status terms such as "experimental",
should be avoided entirely. To avoid:

.. code-block:: none

   The charm should not yet be used in the following scenarios...

   ... but note that this is an experimental feature.

Release notes are an exception as the temporal context is understood:

.. code-block:: none

   The charm now supports feature X...

If at all possible, simply give numbers (e.g. versions, dates) to guide a
reader, but do not hardcode versions into instructions. Explain with words and
include versions only as part of an example command.

.. important::

   If transient information is categorically needed then express it with an
   admonishment. The use of admonishments also makes temporal information much
   easier to identify during documentation reviews.

Whitespace
----------

All extra whitespace should be removed, especially at the end of lines.

.. warning::

   Two trailing spaces is valid Markdown; it forces a carriage return. This is
   very rarely required and should be avoided whenever possible.

To check a file for trailing spaces (tested with Bash and Zsh):

.. code-block:: none

   grep -n "[[:space:]]$" <file>

To view whitespace with the Vim editor, edit ``~/.vimrc``:

.. code-block:: none

   set listchars=tab:>-,trail:·,eol:$
   nmap <silent> <leader>w :set nolist!<CR>

The default leader character is the backslash, so toggle your whitespace
goggles with :command:`\\w` while in command mode.

Snippets
--------

Some messaging is used repeatedly due to situations that arise regularly. This
section is an attempt at making a consistent set of snippets for such cases.
Use the appropriate RST or MD formatting.

Preview charms or functionality
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Use an informational admonishment to convey tech-preview status for a charm, or
functionality for an existing charm:

.. code-block:: none

   The MySQL 8 charms are in a tech-preview state and are ready for testing.
   They are not production-ready.

   Charmed Swift global cluster functionality is in a tech-preview state and is
   ready for testing. It is not production-ready.

Version requirements or limitations
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Use an informational admonishment to convey a software requirement or
limitation for a charm, or functionality for an existing charm:

.. code-block:: none

   BlueStore compression is supported starting with Ceph Mimic.

Command syntax
--------------

The following :command:`deploy` or :command:`add-unit` command syntax and
ordering of options should be observed:

.. code-block:: none

   juju deploy -n <X> --to <Y> --config <option=Z> ...

   juju add-unit -n <X> --to <Y> --config <option=Z> ...

Multi-line commands should have their extra lines indented by three spaces:

.. code-block:: none

   openstack role add --user 1ea06b07c73149ca9c6753e07c30383a \
      --project Project1 Member

reStructuredText formatting
---------------------------

RST - General formatting
~~~~~~~~~~~~~~~~~~~~~~~~

Italics - use single asterisks:

.. code-block:: none

   *this is in italics*

Bold - use double asterisks:

.. code-block:: none

   **this is in bold**

Monospace - use double backticks:

.. code-block:: none

   the ``--force`` option may help

RST - Section headers
~~~~~~~~~~~~~~~~~~~~~

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

RST - Inline commands
~~~~~~~~~~~~~~~~~~~~~

For commands or utilities that are mentioned in a sentence use the
``:command:`` directive:

.. code-block:: none

   You can type the :command:`juju status` command to get an overview of the
   model.

   The :command:`openstack` client is the preferred tool.

RST - Linking to an external site
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

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

RST - Linking to a page in the doc set
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Assuming that the destination document is ``install-maas.rst`` then in the
source document:

.. code-block:: none

   In the :doc:`previous section <install-maas>`

The linking is relative. If the destination document was in the parent
directory:

.. code-block:: none

   In the :doc:`previous section <../install-maas>`

RST - Linking to a location within the current page
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

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

RST - Linking to a location within a page in the doc set
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In the source document:

.. code-block:: none

   during the :ref:`Install MAAS <install_maas>` step on the previous page

In the destination document insert the target code (typically above a section
header):

.. code-block:: none

   .. _install_maas:

   Install MAAS
   ------------

RST - Admonishments
~~~~~~~~~~~~~~~~~~~

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

RST - Code blocks
~~~~~~~~~~~~~~~~~

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

console input
^^^^^^^^^^^^^

.. code-block:: none

   The following command shows the relations:

   .. code-block:: none

      juju status --relations

console output
^^^^^^^^^^^^^^

.. code-block:: none

   Sample output of the last command is:

   .. code-block:: console

      Name            Version      Rev    Tracking        Publisher    Notes
      charm           2.8.2        609    latest/stable   canonical✓   classic
      charmcraft      1.4.0        761    latest/stable   canonical✓   classic

code snippet
^^^^^^^^^^^^

.. code-block:: none

   This bit of Python will do the trick:

   .. code-block:: python

      def anagram(first, second):
       return Counter(first) == Counter(second)

Do not use the ``bash`` type for simple command invocations.

miscellaneous file contents
^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: none

   The contents of file ``/etc/ec2_version`` is:

   .. code-block:: ini

      Ubuntu 20.04.1 LTS (Focal)

RST - Lists
~~~~~~~~~~~

Add a blank line between each item if any list items are multi-lined.

Unordered list
^^^^^^^^^^^^^^

.. code-block:: none

   * First item. Align any word-wrapped lines
     like this.

   * Second item

Nested unordered list
^^^^^^^^^^^^^^^^^^^^^

For nested lists, indent items so they align with the parent text:

.. code-block:: none

   * First item

     * Nested item
     * Nested item

   * First item

     * Nested item
     * Nested item

Ordered list
^^^^^^^^^^^^

.. code-block:: none

   #. First item
   #. Second item

Nested ordered list
^^^^^^^^^^^^^^^^^^^

For nested lists, indent items so they align with the parent text:

.. code-block:: none

   #. First item

      #. Nested item
      #. Nested item

   #. First item

      #. Nested item
      #. Nested item

RST - Definitions
~~~~~~~~~~~~~~~~~

To define a term, indent its definition by two spaces:

.. code-block:: none

   Charm upgrade
     An upgrade of the charm software which is used to deploy and manage
     OpenStack. This includes charms that manage applications which are not
     technically part of the OpenStack project.

RST - Images and figures
~~~~~~~~~~~~~~~~~~~~~~~~

To insert an image or a figure:

.. code-block:: none

   .. image:: <relative/path/to/image.png>
      :<property>
      :<property>

See `RST documentation on images and figures`_ for details.

Markdown formatting
-------------------

MD - General formatting
~~~~~~~~~~~~~~~~~~~~~~~

Italics - use single asterisks:

.. code-block:: none

   *this is in italics*

Bold - use double asterisks:

.. code-block:: none

   **this is in bold**

Monospace - use single backticks:

.. code-block:: none

   the `--force` option may help

MD - Section headers
~~~~~~~~~~~~~~~~~~~~

There are five section headers:

.. code-block:: none

   # H1

   ## H2

   ### H3

   #### H4

   ##### H5

MD - Inline commands
~~~~~~~~~~~~~~~~~~~~

For commands or utilities that are mentioned in a sentence use monospace:

.. code-block:: none

   You can type the `juju status` command to get an overview of the model.

   The `openstack` client is the preferred tool.

MD - Linking to an external site
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: none

   The [OpenStack Charms Deployment Guide][cdg] ...

   .
   .

   ... in the [OpenStack Charm Guide][cg] ...

   .
   .

   See bug [LP #1862392][lp-bug-1862392] ...

   .
   .
   <bottom of page>

   <!-- LINKS -->

   [cg]: https://docs.openstack.org/charm-guide
   [cdg]: https://docs.openstack.org/project-deploy-guide/charm-deployment-guide
   [lp-bug-1862392]: https://bugs.launchpad.net/charm-cinder/+bug/1862392

MD - Linking to a header within the current page
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: none

   See section [Availability zones][anchor-az]...

   .
   .

   ## Availability zones

   .
   .
   <bottom of page>

   <!-- LINKS -->

   [anchor-az]: #availability-zones

MD - Admonishments
~~~~~~~~~~~~~~~~~~

Markdown itself does not have admonishment types as such. Implement an
equivalent RST admonishment as a Markdown quote:

+---------------+-----------------------------------------------+
| Type          | Purpose                                       |
+===============+===============================================+
| ``Tip``       | to provide auxiliary information              |
+---------------+-----------------------------------------------+
| ``Note``      | to inform                                     |
+---------------+-----------------------------------------------+
| ``Important`` | to accentuate                                 |
+---------------+-----------------------------------------------+
| ``Caution``   | to draw special attention to                  |
+---------------+-----------------------------------------------+
| ``Warning``   | to warn about potential breakage or data loss |
+---------------+-----------------------------------------------+

Syntax:

.. code-block:: none

   > **<type>**: text goes here. text goes here. text goes here. text goes here
     maintain the alignment.

The text is left-aligned with the asterisks.

Example:

.. code-block:: none

   > **Note**: The 'ceph-rbd-mirror' charm addresses only one specific element
     in datacentre redundancy.

MD - Code blocks
~~~~~~~~~~~~~~~~

console input
^^^^^^^^^^^^^

Indent four spaces:

.. code-block:: none

   The following command shows the relations:

       juju status --relations

console output
^^^^^^^^^^^^^^

Indent four spaces:

.. code-block:: none

   Sample output of the last command is:

       Name              Version               Rev    Tracking        Publisher    Notes
       charm             2.8.2                 609    latest/stable   canonical✓   classic
       charmcraft        1.4.0                 761    latest/stable   canonical✓   classic

code snippet
^^^^^^^^^^^^

Use syntax highlighting for code snippets/scripts using backticks and a
language type:

* ``python``
* ``bash``
* ``yaml``

Do not use the ``bash`` type for simple command invocations.

Example:

.. code-block:: none

   This bit of Python will do the trick:

   ```python
      import random

      def flip():
          if random.randint(0,1) == 0:
              return "heads"
          else:
              return "tails"
            def anagram(first, second):
             return Counter(first) == Counter(second)
   ```

Use your prerogative for indentation.

miscellaneous file contents
^^^^^^^^^^^^^^^^^^^^^^^^^^^

Indent file contents with four spaces:

.. code-block:: none

   The contents of file ``/etc/ec2_version`` is:

       Ubuntu 20.04.1 LTS (Focal)

MD - Lists
~~~~~~~~~~

Add a blank line between each item if any list items are multi-lined.

Unordered list
^^^^^^^^^^^^^^

.. code-block:: none

   * First item. Align any word-wrapped lines
     like this.

   * Second item

Nested unordered list
^^^^^^^^^^^^^^^^^^^^^

Indent nested items with four spaces:

.. code-block:: none

   * First item
       * Nested item

Ordered list
^^^^^^^^^^^^

.. code-block:: none

   1. First item
   1. Second item

Nested ordered list
^^^^^^^^^^^^^^^^^^^

Indent nested items with four spaces:

.. code-block:: none

   1. First item
       1. Nested item

MD - Images
~~~~~~~~~~~

A regular image:

.. code-block:: none

   ![alt-text][image]

   .
   .

   <bottom of page>

   <!-- LINKS -->

   [image]: path to image

An image as hyperlink:

.. code-block:: none

   [![alt-text][image]][image-target-link]

   .
   .

   <bottom of page>

   <!-- LINKS -->

   [image]: path to image
   [image-target-link]: link URL

.. LINKS
.. _OpenStack Charm Guide: https://docs.openstack.org/charm-guide/latest/
.. _OpenStack Charms Deployment Guide: https://docs.openstack.org/project-deploy-guide/charm-deployment-guide/latest/
.. _OpenStack charms: https://github.com/orgs/openstack-charmers/repositories?q=charm-&type=&language=&sort=
.. _Canonical documentation style guide: https://docs.ubuntu.com/styleguide/en
.. _OpenStack documentation contributor guide: https://docs.openstack.org/doc-contrib-guide
.. _Sphinx: https://www.sphinx-doc.org/en/master/index.html
.. _reStructuredText: https://www.sphinx-doc.org/en/master/usage/restructuredtext/index.html
.. _RST documentation on images and figures: https://docutils.sourceforge.io/docs/ref/rst/directives.html#images
