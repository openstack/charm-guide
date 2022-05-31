===================
Ceph erasure coding
===================

Overview
--------

Ceph pools supporting applications within an OpenStack deployment are
by default configured as replicated pools which means that every stored
object is copied to multiple hosts or zones to allow the pool to survive
the loss of an OSD.

Ceph also supports Erasure Coded pools which can be used to save
raw space within the Ceph cluster.  The following charms can be configured
to use Erasure Coded pools:

* `ceph-fs`_
* `ceph-radosgw`_
* `cinder-ceph`_
* `glance`_
* `nova-compute`_

.. warning::

   Enabling the use of Erasure Coded pools will effect the IO performance
   of the pool and will incur additional CPU and memory overheads on the
   Ceph OSD nodes due to calculation of coding chunks during read and
   write operations and during recovery of data chunks from failed OSDs.

.. note::

   The mirroring of RBD images stored in Erasure Coded pools is not currently
   supported by the ceph-rbd-mirror charm due to limitations in the functionality
   of the Ceph rbd-mirror application.

Configuring charms for Erasure Coding
-------------------------------------

Charms that support Erasure Coded pools have a consistent set of configuration
options to enable and tune the Erasure Coding profile used to configure
the Erasure Coded pools created for each application.

Erasure Coding is enabled by setting the ``pool-type`` option to 'erasure-coded'.

Ceph supports multiple `Erasure Code`_ plugins. A plugin may provide support for
multiple Erasure Code techniques - for example the JErasure plugin provides
support for Cauchy and Reed-Solomon Vandermonde (and others).

For the default JErasure plugin, the K value defines the number of data chunks
that will be used for each object and the M value defines the number of coding
chunks generated for each object. The M value also defines the number of hosts
or zones that may be lost before the pool goes into a degraded state.

K + M must always be less than or equal to the number of hosts or zones in the
deployment (depending on the configuration of ``customize-failure-domain``).

By default the JErasure plugin is used with K=1 and M=2.  This does not
actually save any raw storage compared to a replicated pool with 3 replicas
(and is to allow use on a three node Ceph cluster) so most deployments
using Erasure Coded pools will need to tune the K and M values based on either
the number of hosts deployed or the number of zones in the deployment (if
the ``customize-failure-domain`` option is enabled on the ceph-osd and ceph-mon
charms).

In the example below, the Erasure Coded pool used by the glance application
will sustain the loss of two hosts or zones while only consuming 2TB instead
of 3TB of storage to store 1TB of data when compared to a replicated pool. This
configuration requires a minimum of 4 hosts (or zones).

.. code-block:: yaml

    glance:
      options:
        pool-type: erasure-coded
        ec-profile-k: 2
        ec-profile-m: 2

The full list of Erasure Coding configuration options is detailed below.
Full descriptions of each plugin and its configuration options can also
be found in the `Ceph Erasure Code`_ documention for the Ceph project.

.. list-table:: Erasure Coding charm options
   :widths: 20 15 5 15 45
   :header-rows: 1

   * - Option
     - Charm
     - Type
     - Default Value
     - Description
   * - pool-type
     - all
     - string
     - replicated
     - Ceph pool type to use for storage - valid values are 'replicated' and 'erasure-coded'.
   * - ec-rbd-metadata-pool
     - glance, cinder-ceph, nova-compute
     - string
     -
     - Name of the metadata pool to be created (for RBD use-cases). If not defined a metadata pool name will be generated based on the name of the data pool used by the application.  The metadata pool is always replicated (not erasure coded).
   * - metadata-pool
     - ceph-fs
     - string
     -
     - Name of the metadata pool to be created for the CephFS filesystem. If not defined a metadata pool name will be generated based on the name of the data pool used by the application.  The metadata pool is always replicated (not erasure coded).
   * - ec-profile-name
     - all
     - string
     -
     - Name for the EC profile to be created for the EC pools. If not defined a profile name will be generated based on the name of the pool used by the application.
   * - ec-profile-k
     - all
     - int
     - 1
     - Number of data chunks that will be used for EC data pool. K+M factors should never be greater than the number of available AZs for balancing.
   * - ec-profile-m
     - all
     - int
     - 2
     - Number of coding chunks that will be used for EC data pool. K+M factors should never be greater than number of available AZs for balancing.
   * - ec-profile-locality
     - all
     - int
     -
     - (lrc plugin - l) Group the coding and data chunks into sets of size l. For instance, for k=4 and m=2, when l=3 two groups of three are created. Each set can be recovered without reading chunks from another set.  Note that using the lrc plugin does incur more raw storage usage than isa or jerasure in order to reduce the cost of recovery operations.
   * - ec-profile-crush-locality
     - all
     - string
     -
     - (lrc plugin) The type of the crush bucket in which each set of chunks defined by l will be stored. For instance, if it is set to rack, each group of l chunks will be placed in a different rack. It is used to create a CRUSH rule step such as 'step choose rack'. If it is not set, no such grouping is done.
   * - ec-profile-durability-estimator
     - all
     - int
     -
     - (shec plugin - c) The number of parity chunks each of which includes each data chunk in its calculation range. The number is used as a durability estimator. For instance, if c=2, 2 OSDs can be down without losing data.
   * - ec-profile-helper-chunks
     - all
     - int
     -
     - (clay plugin - d) Number of OSDs requested to send data during recovery of a single chunk. d needs to be chosen such that k+1 <= d <= k+m-1. The larger the d, the better the savings.
   * - ec-profile-scalar-mds
     - all
     - string
     -
     - (clay plugin) Specifies the plugin that is used as a building block in the layered construction. It can be one of: jerasure, isa or shec.
   * - ec-profile-plugin
     - all
     - string
     - jerasure
     - EC plugin to use for this applications pool. These plugins are available: jerasure, lrc, isa, shec, clay.
   * - ec-profile-technique
     - all
     - string
     - reed_sol_van
     - EC profile technique used for this applications pool - will be validated based on the plugin configured via ec-profile-plugin. Supported techniques are 'reed_sol_van', 'reed_sol_r6_op', 'cauchy_orig', 'cauchy_good', 'liber8tion' for jerasure, 'reed_sol_van', 'cauchy' for isa and 'single', 'multiple' for shec.
   * - ec-profile-device-class
     - all
     - string
     -
     - Device class from CRUSH map to use for placement groups for erasure profile - valid values: ssd, hdd or nvme (or leave unset to not use a device class).


Ceph automatic device classing
------------------------------

Newer versions of Ceph perform automatic classing of OSD devices. Each OSD
will be placed into ‘nvme’, ‘ssd’ or ‘hdd’ device classes.  These can
be used when enabling Erasure Coded pools.

Device classes can be inspected using:

.. code::

    sudo ceph osd crush tree

    ID CLASS WEIGHT  TYPE NAME
    -1       8.18729 root default
    -5       2.72910     host node-laveran
     2  nvme 0.90970         osd.2
     5   ssd 0.90970         osd.5
     7   ssd 0.90970         osd.7
    -7       2.72910     host node-mees
     1  nvme 0.90970         osd.1
     6   ssd 0.90970         osd.6
     8   ssd 0.90970         osd.8
    -3       2.72910     host node-pytheas
     0  nvme 0.90970         osd.0
     3   ssd 0.90970         osd.3
     4   ssd 0.90970         osd.4

The device class for an Erasure Coded pool can be configured in the
consuming charm using the ``ec-device-class`` configuration option.

If this option is not provided devices of any class will be used.

.. LINKS
.. _Ceph Erasure Code: https://docs.ceph.com/docs/master/rados/operations/erasure-code/
.. _ceph-fs: https://jaas.ai/ceph-fs
.. _ceph-radosgw: https://jaas.ai/ceph-radosgw
.. _cinder-ceph: https://jaas.ai/cinder-ceph
.. _glance: https://jaas.ai/glance
.. _nova-compute: https://jaas.ai/nova-compute
.. _Erasure Code: https://en.wikipedia.org/wiki/Erasure_code
