==================================
Pre-channel charm revision numbers
==================================

This page lists the most recent revision numbers for the
:doc:`../release-notes/2110` charms release - the last release of legacy
(non-channel) charms.

The information presented here may be of use to administrators whose
deployments are not running channel type charms or that are using Juju versions
(< 2.9) that are incompatible with channel type charms. This is due to the fact
that the OpenStack Charms project no longer supports the ``latest`` track,
which historically was a very common means to consume charms in the pre-channel
era.

The following is recommended reading:

* :doc:`../concepts/charm-types`
* :doc:`../project/charm-delivery`

Revision numbers
----------------

The below tables list the charms, series, and last known revision number (and
architecture, if applicable). Information is categorised by charm groups:

* OpenStack charms
* Ceph charms
* OVN charms
* miscellaneous support charms

To use these revisions, the charm needs to be downloaded and then deployed
locally. Furthermore, the download must be done with version 2.x of the
charm-tools snap.

For example, to deploy revision 46 of the aodh charm:

.. code-block:: none

   sudo snap install charm --channel=2.x/stable --classic
   cd <some-temporary-dir>
   charm pull aodh-46
   juju deploy ./aodh


OpenStack
~~~~~~~~~

.. tabs::

   .. group-tab:: aodh

      .. list-table::
         :header-rows: 1
         :widths: 1 1 1 1 1 1 1 1 1 1 1 1 1 1
         :width: 75%
         :stub-columns: 0

         * - arch
           - 14.04
           - 16.04
           - 16.10
           - 17.04
           - 17.10
           - 18.04
           - 18.10
           - 19.04
           - 19.10
           - 20.04
           - 20.10
           - 21.04
           - 21.10

         * - all
           - 46
           - 57
           - 6
           - 10
           - 14
           - 57
           - 33
           - 39
           - 44
           - 57
           - 57
           - 57
           - 57

   .. group-tab:: barbican

      .. list-table::
         :header-rows: 1
         :widths: 1 1 1 1 1 1 1 1 1 1 1
         :width: 75%
         :stub-columns: 0

         * - arch
           - 16.04
           - 17.10
           - 18.04
           - 18.10
           - 19.04
           - 19.10
           - 20.04
           - 20.10
           - 21.04
           - 21.10

         * - all
           - 9
           - 5
           - 46
           - 22
           - 29
           - 34
           - 46
           - 46
           - 46
           - 46

   .. group-tab:: barbican-vault

      .. list-table::
         :header-rows: 1
         :widths: 1 1 1 1 1 1 1 1 1
         :width: 75%
         :stub-columns: 0

         * - arch
           - 18.04
           - 18.10
           - 19.04
           - 19.10
           - 20.04
           - 20.10
           - 21.04
           - 21.10

         * - all
           - 27
           - 7
           - 9
           - 18
           - 27
           - 27
           - 27
           - 27

   .. group-tab:: ceilometer

      .. list-table::
         :header-rows: 1
         :widths: 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1
         :width: 75%
         :stub-columns: 0

         * - arch
           - 12.04
           - 14.04
           - 16.04
           - 16.10
           - 17.04
           - 17.10
           - 18.04
           - 18.10
           - 19.04
           - 19.10
           - 20.04
           - 20.10
           - 21.04
           - 21.10

         * - all
           - 439
           - 478
           - 489
           - 445
           - 448
           - 452
           - 489
           - 464
           - 466
           - 476
           - 489
           - 489
           - 489
           - 489

   .. group-tab:: ceilometer-agent

      .. list-table::
         :header-rows: 1
         :widths: 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1
         :width: 75%
         :stub-columns: 0

         * - arch
           - 12.04
           - 14.04
           - 16.04
           - 16.10
           - 17.04
           - 17.10
           - 18.04
           - 18.10
           - 19.04
           - 19.10
           - 20.04
           - 20.10
           - 21.04
           - 21.10

         * - all
           - 433
           - 466
           - 474
           - 436
           - 438
           - 442
           - 474
           - 453
           - 458
           - 464
           - 474
           - 474
           - 474
           - 474

.. tabs::

   .. group-tab:: cinder

      .. list-table::
         :header-rows: 1
         :widths: 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1
         :width: 75%
         :stub-columns: 0

         * - arch
           - 12.04
           - 14.04
           - 16.04
           - 16.10
           - 17.04
           - 17.10
           - 18.04
           - 18.10
           - 19.04
           - 19.10
           - 20.04
           - 20.10
           - 21.04
           - 21.10

         * - all
           - 470
           - 519
           - 530
           - 475
           - 477
           - 484
           - 530
           - 501
           - 512
           - 517
           - 530
           - 530
           - 530
           - 530

   .. group-tab:: cinder-backup

      .. list-table::
         :header-rows: 1
         :widths: 1 1 1 1 1 1 1 1 1 1 1 1 1 1
         :width: 75%
         :stub-columns: 0

         * - arch
           - 12.04
           - 14.04
           - 16.04
           - 16.10
           - 17.04
           - 17.10
           - 18.04
           - 18.10
           - 19.10
           - 20.04
           - 20.10
           - 21.04
           - 21.10

         * - all
           - 12
           - 31
           - 37
           - 15
           - 17
           - 21
           - 37
           - 25
           - 29
           - 37
           - 37
           - 37
           - 37

   .. group-tab:: cinder-ceph

      .. list-table::
         :header-rows: 1
         :widths: 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1
         :width: 75%
         :stub-columns: 0

         * - arch
           - 12.04
           - 14.04
           - 16.04
           - 16.10
           - 17.04
           - 17.10
           - 18.04
           - 18.10
           - 19.04
           - 19.10
           - 20.04
           - 20.10
           - 21.04
           - 21.10

         * - all
           - 436
           - 475
           - 483
           - 441
           - 443
           - 448
           - 483
           - 464
           - 466
           - 473
           - 483
           - 483
           - 483
           - 483

   .. group-tab:: openstack-dashboard

      .. list-table::
         :header-rows: 1
         :widths: 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1
         :width: 75%
         :stub-columns: 0

         * - arch
           - 12.04
           - 14.04
           - 16.04
           - 16.10
           - 17.04
           - 17.10
           - 18.04
           - 18.10
           - 19.04
           - 19.10
           - 20.04
           - 20.10
           - 21.04
           - 21.10

         * - all
           - 442
           - 507
           - 514
           - 448
           - 451
           - 460
           - 514
           - 489
           - 498
           - 505
           - 514
           - 514
           - 514
           - 514

   .. group-tab:: designate

      .. list-table::
         :header-rows: 1
         :widths: 1 1 1 1 1 1 1 1 1 1 1 1 1 1
         :width: 75%
         :stub-columns: 0

         * - arch
           - 14.04
           - 16.04
           - 16.10
           - 17.04
           - 17.10
           - 18.04
           - 18.10
           - 19.04
           - 19.10
           - 20.04
           - 20.10
           - 21.04
           - 21.10

         * - all
           - 51
           - 60
           - 9
           - 14
           - 20
           - 60
           - 33
           - 42
           - 49
           - 60
           - 60
           - 60
           - 60

.. tabs::

   .. group-tab:: designate-bind

      .. list-table::
         :header-rows: 1
         :widths: 1 1 1 1 1 1 1 1 1 1 1 1 1 1
         :width: 75%
         :stub-columns: 0

         * - arch
           - 14.04
           - 16.04
           - 16.10
           - 17.04
           - 17.10
           - 18.04
           - 18.10
           - 19.04
           - 19.10
           - 20.04
           - 20.10
           - 21.04
           - 21.10

         * - all
           - 34
           - 43
           - 7
           - 10
           - 13
           - 43
           - 23
           - 27
           - 32
           - 43
           - 43
           - 43
           - 43

   .. group-tab:: glance

      .. list-table::
         :header-rows: 1
         :widths: 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1
         :width: 75%
         :stub-columns: 0

         * - arch
           - 12.04
           - 14.04
           - 16.04
           - 16.10
           - 17.04
           - 17.10
           - 18.04
           - 18.10
           - 19.04
           - 19.10
           - 20.04
           - 20.10
           - 21.04
           - 21.10

         * - all
           - 459
           - 508
           - 516
           - 463
           - 465
           - 472
           - 516
           - 492
           - 500
           - 506
           - 516
           - 516
           - 516
           - 516

   .. group-tab:: glance-simplestreams-sync

      .. list-table::
         :header-rows: 1
         :widths: 1 1 1 1 1 1 1 1 1 1 1 1
         :width: 75%
         :stub-columns: 0

         * - arch
           - 14.04
           - 16.04
           - 16.10
           - 18.04
           - 18.10
           - 19.04
           - 19.10
           - 20.04
           - 20.10
           - 21.04
           - 21.10

         * - all
           - 33
           - 47
           - 11
           - 47
           - 23
           - 25
           - 28
           - 47
           - 47
           - 47
           - 47

   .. group-tab:: gnocchi

      .. list-table::
         :header-rows: 1
         :widths: 1 1 1 1 1 1 1 1 1 1 1 1 1
         :width: 75%
         :stub-columns: 0

         * - arch
           - 14.04
           - 16.04
           - 17.04
           - 17.10
           - 18.04
           - 18.10
           - 19.04
           - 19.10
           - 20.04
           - 20.10
           - 21.04
           - 21.10

         * - all
           - 7
           - 51
           - 5
           - 10
           - 51
           - 26
           - 37
           - 42
           - 51
           - 51
           - 51
           - 51

   .. group-tab:: heat

      .. list-table::
         :header-rows: 1
         :widths: 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1
         :width: 75%
         :stub-columns: 0

         * - arch
           - 12.04
           - 14.04
           - 16.04
           - 16.10
           - 17.04
           - 17.10
           - 18.04
           - 18.10
           - 19.04
           - 19.10
           - 20.04
           - 20.10
           - 21.04
           - 21.10

         * - all
           - 436
           - 479
           - 485
           - 440
           - 444
           - 452
           - 485
           - 467
           - 472
           - 477
           - 485
           - 485
           - 485
           - 485

.. tabs::

   .. group-tab:: keystone

      .. list-table::
         :header-rows: 1
         :widths: 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1
         :width: 75%
         :stub-columns: 0

         * - arch
           - 12.04
           - 14.04
           - 16.04
           - 16.10
           - 17.04
           - 17.10
           - 18.04
           - 18.10
           - 19.04
           - 19.10
           - 20.04
           - 20.10
           - 21.04
           - 21.10

         * - all
           - 473
           - 530
           - 539
           - 478
           - 483
           - 493
           - 539
           - 515
           - 521
           - 528
           - 539
           - 539
           - 539
           - 539

   .. group-tab:: keystone-ldap

      .. list-table::
         :header-rows: 1
         :widths: 1 1 1 1 1 1 1 1 1 1 1 1 1 1
         :width: 75%
         :stub-columns: 0

         * - arch
           - 14.04
           - 16.04
           - 16.10
           - 17.04
           - 17.10
           - 18.04
           - 18.10
           - 19.04
           - 19.10
           - 20.04
           - 20.10
           - 21.04
           - 21.10

         * - all
           - 33
           - 42
           - 4
           - 6
           - 10
           - 42
           - 21
           - 27
           - 31
           - 42
           - 42
           - 42
           - 42

   .. group-tab:: manila

      .. list-table::
         :header-rows: 1
         :widths: 1 1 1 1 1 1 1 1 1 1 1 1
         :width: 75%
         :stub-columns: 0

         * - arch
           - 16.04
           - 16.10
           - 17.04
           - 17.10
           - 18.04
           - 18.10
           - 19.10
           - 20.04
           - 20.10
           - 21.04
           - 21.10

         * - all
           - 36
           - 0
           - 8
           - 11
           - 36
           - 16
           - 25
           - 36
           - 36
           - 36
           - 36

   .. group-tab:: manila-ganesha

      .. list-table::
         :header-rows: 1
         :widths: 1 1 1 1 1 1 1 1
         :width: 75%
         :stub-columns: 0

         * - arch
           - 18.04
           - 19.04
           - 19.10
           - 20.04
           - 20.10
           - 21.04
           - 21.10

         * - all
           - 20
           - 6
           - 6
           - 20
           - 20
           - 20
           - 20

   .. group-tab:: masakari

      .. list-table::
         :header-rows: 1
         :widths: 1 1 1 1 1 1 1
         :width: 75%
         :stub-columns: 0

         * - arch
           - 18.04
           - 19.10
           - 20.04
           - 20.10
           - 21.04
           - 21.10

         * - all
           - 17
           - 6
           - 17
           - 17
           - 17
           - 17

.. tabs::

   .. group-tab:: masakari-monitors

      .. list-table::
         :header-rows: 1
         :widths: 1 1 1 1 1 1 1
         :width: 75%
         :stub-columns: 0

         * - arch
           - 18.04
           - 19.10
           - 20.04
           - 20.10
           - 21.04
           - 21.10

         * - all
           - 14
           - 4
           - 14
           - 14
           - 14
           - 14

   .. group-tab:: neutron-api

      .. list-table::
         :header-rows: 1
         :widths: 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1
         :width: 75%
         :stub-columns: 0

         * - arch
           - 12.04
           - 14.04
           - 15.10
           - 16.04
           - 16.10
           - 17.04
           - 17.10
           - 18.04
           - 18.10
           - 19.04
           - 19.10
           - 20.04
           - 20.10
           - 21.04
           - 21.10

         * - all
           - 446
           - 489
           - 7
           - 501
           - 450
           - 454
           - 460
           - 501
           - 475
           - 481
           - 487
           - 501
           - 501
           - 501
           - 501

   .. group-tab:: neutron-gateway

      .. list-table::
         :header-rows: 1
         :widths: 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1
         :width: 75%
         :stub-columns: 0

         * - arch
           - 12.04
           - 14.04
           - 16.04
           - 16.10
           - 17.04
           - 17.10
           - 18.04
           - 18.10
           - 19.04
           - 19.10
           - 20.04
           - 20.10
           - 21.04
           - 21.10

         * - all
           - 431
           - 480
           - 488
           - 436
           - 443
           - 451
           - 488
           - 464
           - 471
           - 477
           - 488
           - 488
           - 488
           - 488

   .. group-tab:: neutron-openvswitch

      .. list-table::
         :header-rows: 1
         :widths: 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1
         :width: 75%
         :stub-columns: 0

         * - arch
           - 12.04
           - 14.04
           - 16.04
           - 16.10
           - 17.04
           - 17.10
           - 18.04
           - 18.10
           - 19.04
           - 19.10
           - 20.04
           - 20.10
           - 21.04
           - 21.10

         * - all
           - 437
           - 479
           - 493
           - 442
           - 445
           - 449
           - 493
           - 466
           - 473
           - 477
           - 493
           - 493
           - 493
           - 493

   .. group-tab:: nova-cloud-controller

      .. list-table::
         :header-rows: 1
         :widths: 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1
         :width: 75%
         :stub-columns: 0

         * - arch
           - 12.04
           - 14.04
           - 15.10
           - 16.04
           - 16.10
           - 17.04
           - 17.10
           - 18.04
           - 18.10
           - 19.04
           - 19.10
           - 20.04
           - 20.10
           - 21.04
           - 21.10

         * - all
           - 495
           - 556
           - 4
           - 566
           - 502
           - 505
           - 513
           - 566
           - 535
           - 546
           - 553
           - 566
           - 566
           - 566
           - 566

.. tabs::

   .. group-tab:: nova-compute

      .. list-table::
         :header-rows: 1
         :widths: 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1
         :width: 75%
         :stub-columns: 0

         * - arch
           - 12.04
           - 14.04
           - 16.04
           - 16.10
           - 17.04
           - 17.10
           - 18.04
           - 18.10
           - 19.04
           - 19.10
           - 20.04
           - 20.10
           - 21.04
           - 21.10

         * - all
           - 475
           - 538
           - 550
           - 485
           - 492
           - 499
           - 550
           - 520
           - 527
           - 536
           - 550
           - 550
           - 550
           - 550

   .. group-tab:: nova-cell-controller

      .. list-table::
         :header-rows: 1
         :widths: 1 1 1 1 1 1 1
         :width: 75%
         :stub-columns: 0

         * - arch
           - 18.04
           - 19.10
           - 20.04
           - 20.10
           - 21.04
           - 21.10

         * - all
           - 16
           - 3
           - 16
           - 16
           - 16
           - 16

   .. group-tab:: octavia

      .. list-table::
         :header-rows: 1
         :widths: 1 1 1 1 1 1 1 1 1
         :width: 75%
         :stub-columns: 0

         * - arch
           - 18.04
           - 18.10
           - 19.04
           - 19.10
           - 20.04
           - 20.10
           - 21.04
           - 21.10

         * - all
           - 39
           - 11
           - 20
           - 32
           - 39
           - 39
           - 39
           - 39

   .. group-tab:: placement

      .. list-table::
         :header-rows: 1
         :widths: 1 1 1 1 1 1 1
         :width: 75%
         :stub-columns: 0

         * - arch
           - 18.04
           - 19.10
           - 20.04
           - 20.10
           - 21.04
           - 21.10

         * - all
           - 32
           - 15
           - 32
           - 32
           - 32
           - 32

   .. group-tab:: swift-proxy

      .. list-table::
         :header-rows: 1
         :widths: 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1
         :width: 75%
         :stub-columns: 0

         * - arch
           - 12.04
           - 14.04
           - 16.04
           - 16.10
           - 17.04
           - 17.10
           - 18.04
           - 18.10
           - 19.04
           - 19.10
           - 20.04
           - 20.10
           - 21.04
           - 21.10

         * - all
           - 69
           - 109
           - 115
           - 72
           - 75
           - 81
           - 115
           - 99
           - 103
           - 107
           - 115
           - 115
           - 115
           - 115

.. tabs::

   .. group-tab:: swift-storage

      .. list-table::
         :header-rows: 1
         :widths: 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1
         :width: 75%
         :stub-columns: 0

         * - arch
           - 12.04
           - 14.04
           - 16.04
           - 16.10
           - 17.04
           - 17.10
           - 18.04
           - 18.10
           - 19.04
           - 19.10
           - 20.04
           - 20.10
           - 21.04
           - 21.10

         * - all
           - 416
           - 455
           - 461
           - 421
           - 423
           - 429
           - 461
           - 444
           - 449
           - 453
           - 461
           - 461
           - 461
           - 461

   .. group-tab:: cinder-backup-swift-proxy

      .. list-table::
         :header-rows: 1
         :widths: 1 1 1 1 1 1
         :width: 75%
         :stub-columns: 0

         * - arch
           - 18.04
           - 20.04
           - 20.10
           - 21.04
           - 21.10

         * - all
           - 13
           - 13
           - 13
           - 13
           - 13

   .. group-tab:: cinder-lvm

      .. list-table::
         :header-rows: 1
         :widths: 1 1 1 1 1 1
         :width: 75%
         :stub-columns: 0

         * - arch
           - 18.04
           - 20.04
           - 20.10
           - 21.04
           - 21.10

         * - all
           - 0
           - 0
           - 0
           - 0
           - 0

   .. group-tab:: cinder-purestorage

      .. list-table::
         :header-rows: 1
         :widths: 1 1 1 1 1 1 1 1 1
         :width: 75%
         :stub-columns: 0

         * - arch
           - 16.04
           - 18.04
           - 19.04
           - 19.10
           - 20.04
           - 20.10
           - 21.04
           - 21.10

         * - all
           - 19
           - 19
           - 1
           - 9
           - 19
           - 19
           - 19
           - 19

   .. group-tab:: cinder-netapp

      .. list-table::
         :header-rows: 1
         :widths: 1 1 1 1 1 1
         :width: 75%
         :stub-columns: 0

         * - arch
           - 18.04
           - 20.04
           - 20.10
           - 21.04
           - 21.10

         * - all
           - 0
           - 0
           - 0
           - 0
           - 0

.. tabs::

   .. group-tab:: keystone-kerberos

      .. list-table::
         :header-rows: 1
         :widths: 1 1 1 1 1 1
         :width: 75%
         :stub-columns: 0

         * - arch
           - 18.04
           - 20.04
           - 20.10
           - 21.04
           - 21.10

         * - all
           - 12
           - 12
           - 12
           - 12
           - 12

   .. group-tab:: keystone-saml-mellon

      .. list-table::
         :header-rows: 1
         :widths: 1 1 1 1 1 1 1 1
         :width: 75%
         :stub-columns: 0

         * - arch
           - 16.04
           - 18.04
           - 19.10
           - 20.04
           - 20.10
           - 21.04
           - 21.10

         * - all
           - 12
           - 12
           - 2
           - 12
           - 12
           - 12
           - 12

   .. group-tab:: magnum

      .. list-table::
         :header-rows: 1
         :widths: 1 1
         :width: 75%
         :stub-columns: 0

         * - arch
           - 20.04

         * - amd64
           - 2

         * - arm64
           - 2

         * - ppc64le
           - 2

         * - s390x
           - 2

   .. group-tab:: magnum-dashboard

      .. list-table::
         :header-rows: 1
         :widths: 1 1
         :width: 75%
         :stub-columns: 0

         * - arch
           - 20.04

         * - amd64
           - 1

         * - arm64
           - 1

         * - ppc64le
           - 1

         * - s390x
           - 1

   .. group-tab:: neutron-api-plugin-ovn

      .. list-table::
         :header-rows: 1
         :widths: 1 1 1 1 1 1 1
         :width: 75%
         :stub-columns: 0

         * - arch
           - 18.04
           - 19.10
           - 20.04
           - 20.10
           - 21.04
           - 21.10

         * - all
           - 10
           - 2
           - 10
           - 10
           - 10
           - 10

.. tabs::

   .. group-tab:: octavia-dashboard

      .. list-table::
         :header-rows: 1
         :widths: 1 1 1 1 1 1 1 1 1
         :width: 75%
         :stub-columns: 0

         * - arch
           - 18.04
           - 18.10
           - 19.04
           - 19.10
           - 20.04
           - 20.10
           - 21.04
           - 21.10

         * - all
           - 28
           - 15
           - 12
           - 19
           - 28
           - 28
           - 28
           - 28

   .. group-tab:: octavia-diskimage-retrofit

      .. list-table::
         :header-rows: 1
         :widths: 1 1 1 1 1 1 1 1 1
         :width: 75%
         :stub-columns: 0

         * - arch
           - 18.04
           - 18.10
           - 19.04
           - 19.10
           - 20.04
           - 20.10
           - 21.04
           - 21.10

         * - all
           - 23
           - 2
           - 7
           - 15
           - 23
           - 23
           - 23
           - 23

Ceph
~~~~

.. tabs::

   .. group-tab:: ceph-dashboard

      .. list-table::
         :header-rows: 1
         :widths: 1 1
         :width: 75%
         :stub-columns: 0

         * - arch
           - 20.04

         * - amd64
           - 2

   .. group-tab:: ceph-fs

      .. list-table::
         :header-rows: 1
         :widths: 1 1 1 1 1 1 1 1 1 1 1 1 1
         :width: 75%
         :stub-columns: 0

         * - arch
           - 16.04
           - 16.10
           - 17.04
           - 17.10
           - 18.04
           - 18.10
           - 19.04
           - 19.10
           - 20.04
           - 20.10
           - 21.04
           - 21.10

         * - all
           - 36
           - 5
           - 9
           - 14
           - 36
           - 18
           - 21
           - 25
           - 36
           - 36
           - 36
           - 36

   .. group-tab:: ceph-iscsi

      .. list-table::
         :header-rows: 1
         :widths: 1 1 1 1 1
         :width: 75%
         :stub-columns: 0

         * - arch
           - 20.04
           - 20.10
           - 21.04
           - 21.10

         * - all
           - 3
           - 3
           - 3
           - 3

   .. group-tab:: ceph-mon

      .. list-table::
         :header-rows: 1
         :widths: 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1
         :width: 75%
         :stub-columns: 0

         * - arch
           - 12.04
           - 14.04
           - 16.04
           - 16.10
           - 17.04
           - 17.10
           - 18.04
           - 18.10
           - 19.04
           - 19.10
           - 20.04
           - 20.10
           - 21.04
           - 21.10

         * - all
           - 19
           - 64
           - 73
           - 23
           - 31
           - 37
           - 73
           - 53
           - 57
           - 62
           - 73
           - 73
           - 73
           - 73

   .. group-tab:: ceph-osd

      .. list-table::
         :header-rows: 1
         :widths: 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1
         :width: 75%
         :stub-columns: 0

         * - arch
           - 12.04
           - 14.04
           - 16.04
           - 16.10
           - 17.04
           - 17.10
           - 18.04
           - 18.10
           - 19.04
           - 19.10
           - 20.04
           - 20.10
           - 21.04
           - 21.10

         * - all
           - 439
           - 505
           - 513
           - 445
           - 453
           - 468
           - 513
           - 489
           - 498
           - 503
           - 513
           - 513
           - 513
           - 513

.. tabs::

   .. group-tab:: ceph-proxy

      .. list-table::
         :header-rows: 1
         :widths: 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1
         :width: 75%
         :stub-columns: 0

         * - arch
           - 12.04
           - 14.04
           - 16.04
           - 16.10
           - 17.04
           - 17.10
           - 18.04
           - 18.10
           - 19.04
           - 19.10
           - 20.04
           - 20.10
           - 21.04
           - 21.10

         * - all
           - 0
           - 39
           - 46
           - 8
           - 10
           - 15
           - 46
           - 26
           - 28
           - 37
           - 46
           - 46
           - 46
           - 46

   .. group-tab:: ceph-radosgw

      .. list-table::
         :header-rows: 1
         :widths: 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1
         :width: 75%
         :stub-columns: 0

         * - arch
           - 12.04
           - 14.04
           - 15.10
           - 16.04
           - 16.10
           - 17.04
           - 17.10
           - 18.04
           - 18.10
           - 19.04
           - 19.10
           - 20.04
           - 20.10
           - 21.04
           - 21.10

         * - all
           - 449
           - 493
           - 12
           - 499
           - 454
           - 457
           - 462
           - 499
           - 480
           - 485
           - 491
           - 499
           - 499
           - 499
           - 499

   .. group-tab:: ceph-rbd-mirror

      .. list-table::
         :header-rows: 1
         :widths: 1 1 1 1 1 1 1 1 1 1
         :width: 75%
         :stub-columns: 0

         * - arch
           - 16.04
           - 18.04
           - 18.10
           - 19.04
           - 19.10
           - 20.04
           - 20.10
           - 21.04
           - 21.10

         * - all
           - 25
           - 25
           - 5
           - 7
           - 13
           - 25
           - 25
           - 25
           - 25

OVN
~~~

.. tabs::

   .. group-tab:: ovn-central

      .. list-table::
         :header-rows: 1
         :widths: 1 1 1 1 1 1 1
         :width: 75%
         :stub-columns: 0

         * - arch
           - 18.04
           - 19.10
           - 20.04
           - 20.10
           - 21.04
           - 21.10

         * - all
           - 16
           - 1
           - 16
           - 16
           - 16
           - 16

   .. group-tab:: ovn-dedicated-chassis

      .. list-table::
         :header-rows: 1
         :widths: 1 1 1 1 1 1 1
         :width: 75%
         :stub-columns: 0

         * - arch
           - 18.04
           - 19.10
           - 20.04
           - 20.10
           - 21.04
           - 21.10

         * - all
           - 18
           - 4
           - 18
           - 18
           - 18
           - 18

   .. group-tab:: ovn-chassis

      .. list-table::
         :header-rows: 1
         :widths: 1 1 1 1 1 1 1
         :width: 75%
         :stub-columns: 0

         * - arch
           - 18.04
           - 19.10
           - 20.04
           - 20.10
           - 21.04
           - 21.10

         * - all
           - 21
           - 3
           - 21
           - 21
           - 21
           - 21

Miscellaneous
~~~~~~~~~~~~~

.. tabs::

   .. group-tab:: hacluster

      .. list-table::
         :header-rows: 1
         :widths: 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1
         :width: 75%
         :stub-columns: 0

         * - arch
           - 12.04
           - 12.10
           - 14.04
           - 16.04
           - 16.10
           - 17.04
           - 17.10
           - 18.04
           - 18.10
           - 19.04
           - 19.10
           - 20.04
           - 20.10
           - 21.04
           - 21.10

         * - all
           - 16
           - 0
           - 76
           - 83
           - 36
           - 40
           - 49
           - 83
           - 62
           - 68
           - 74
           - 83
           - 83
           - 83
           - 83

   .. group-tab:: mysql-innodb-cluster

      .. list-table::
         :header-rows: 1
         :widths: 1 1 1 1 1 1
         :width: 75%
         :stub-columns: 0

         * - arch
           - 19.10
           - 20.04
           - 20.10
           - 21.04
           - 21.10

         * - all
           - 3
           - 15
           - 15
           - 15
           - 15

   .. group-tab:: mysql-router

      .. list-table::
         :header-rows: 1
         :widths: 1 1 1 1 1 1
         :width: 75%
         :stub-columns: 0

         * - arch
           - 19.10
           - 20.04
           - 20.10
           - 21.04
           - 21.10

         * - all
           - 4
           - 15
           - 15
           - 15
           - 15

   .. group-tab:: percona-cluster

      .. list-table::
         :header-rows: 1
         :widths: 1 1 1 1 1 1 1 1 1 1 1
         :width: 75%
         :stub-columns: 0

         * - arch
           - 12.04
           - 14.04
           - 16.04
           - 16.10
           - 17.04
           - 17.10
           - 18.04
           - 18.10
           - 19.04
           - 19.10

         * - all
           - 0
           - 287
           - 302
           - 253
           - 256
           - 268
           - 302
           - 287
           - 288
           - 293

   .. group-tab:: rabbitmq-server

      .. list-table::
         :header-rows: 1
         :widths: 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1
         :width: 75%
         :stub-columns: 0

         * - arch
           - 12.04
           - 14.04
           - 15.10
           - 16.04
           - 16.10
           - 17.04
           - 17.10
           - 18.04
           - 18.10
           - 19.04
           - 19.10
           - 20.04
           - 20.10
           - 21.04
           - 21.10

         * - all
           - 64
           - 113
           - 8
           - 123
           - 69
           - 74
           - 82
           - 123
           - 100
           - 105
           - 111
           - 123
           - 123
           - 123
           - 123

.. tabs::

   .. group-tab:: openstack-charmers-next-woodpecker

      .. list-table::
         :header-rows: 1
         :widths: 1 1 1
         :width: 75%
         :stub-columns: 0

         * - arch
           - 18.04
           - 20.04

         * - all
           - 5
           - 5

   .. group-tab:: vault

      .. list-table::
         :header-rows: 1
         :widths: 1 1 1 1 1 1 1 1 1 1
         :width: 75%
         :stub-columns: 0

         * - arch
           - 16.04
           - 18.04
           - 18.10
           - 19.04
           - 19.10
           - 20.04
           - 20.10
           - 21.04
           - 21.10

         * - all
           - 54
           - 54
           - 29
           - 32
           - 42
           - 54
           - 54
           - 54
           - 54

   .. group-tab:: pacemaker-remote

      .. list-table::
         :header-rows: 1
         :widths: 1 1 1 1 1 1 1
         :width: 75%
         :stub-columns: 0

         * - arch
           - 18.04
           - 19.10
           - 20.04
           - 20.10
           - 21.04
           - 21.10

         * - all
           - 14
           - 3
           - 14
           - 14
           - 14
           - 14
