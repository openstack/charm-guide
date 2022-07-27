===================
Writing style guide
===================

Overview
--------

This guide describes the writing style for the OpenStack Charms documentation
:ref:`sources <doc_sources_summary>`.

.. note::

   The OpenStack Charms project also abides by these guides:

   * the `OpenStack documentation contributor guide`_
   * the `Ubuntu documentation style guide`_

   However, in cases of disagreement or ambiguity, the current document takes
   precedence.

Writing formats
---------------

The below pages provide specific guidance for the two writing formats used with
the OpenStack Charms project:

.. toctree::
   :maxdepth: 1

   rst
   md

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

Snippets
--------

Some messaging is used repeatedly due to situations that arise regularly. This
section is an attempt at making a consistent set of snippets for such cases.

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

.. LINKS
.. _Ubuntu documentation style guide: https://docs.ubuntu.com/styleguide/en
.. _OpenStack documentation contributor guide: https://docs.openstack.org/doc-contrib-guide
.. _Sphinx: https://www.sphinx-doc.org/en/master/index.html
.. _RST documentation on images and figures: https://docutils.sourceforge.io/docs/ref/rst/directives.html#images
