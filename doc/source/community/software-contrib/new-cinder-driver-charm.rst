===================================
Develop Cinder storage driver charm
===================================

.. important::

   This page has been identified as being affected by the breaking changes
   introduced between versions 2.9.x and 3.x of the Juju client. Read
   support note :ref:`juju_29_3x_changes` before continuing.

The example below will walk through the creation of a basic Cinder storage
subordinate charm for the `Openstack Cinder NFS`_ backend. The charm will
use existing `operator framework helpers`_. Once the charm is written it will
be composed using `charmcraft`_.

Before writing a new charm the charm author needs to have a clear idea of
what files and services the charm is going to manage and what dependencies will
be needed for it to perform its designated task.

The Cinder NFS charm created in this example requires an NFS host and export.

Create the skeleton charm
-------------------------

Prerequisites
~~~~~~~~~~~~~

The :command:`charmcraft` tool is needed to build and publish the new charm,
and a virtualenv is used for charm creation.

.. code-block:: none

   sudo snap install charmcraft
   sudo apt install python3.9-venv tox
   venv/bin/pip3 install cookiecutter

Create the charm
~~~~~~~~~~~~~~~~

The OpenStack Charms team has prepared a cookiecutter template to help
create a new Cinder driver charm. Running the cookiecutter will prompt
the user to answer some questions about the charm and does include
defaults.

.. code-block:: console

    venv/bin/cookiecutter https://github.com/openstack-charmers/cinder-storage-backend-template
    driver_name [MyDriver]: NFS
    driver_name_lc [nfs]:
    release [queens]: xena
    additional_package_name []:
    Select driver_install_method:
    1 - None
    2 - PPA
    3 - Juju Resource DEB
    Choose from 1, 2, 3 [1]: 1
    Initialized empty Git repository in /home/chris/code/canonical/cinder-nfs/.git/
    [main (root-commit) 88a54ad] Initial Cookiecutter Commit.
    18 files changed, 573 insertions(+)
    create mode 100644 .gitignore
    create mode 100644 .stestr.conf
    create mode 100644 README.md
    create mode 100644 build-requirements.txt
    create mode 100644 config.yaml
    create mode 100644 metadata.yaml
    create mode 100755 pip.sh
    create mode 100644 requirements.txt
    create mode 100755 src/charm.py
    create mode 100644 src/test-requirements.txt
    create mode 100644 src/tox.ini
    create mode 100644 test-requirements.txt
    create mode 100644 tests/bundles/bionic-xena.yaml
    create mode 100644 tests/tests.yaml
    create mode 100644 tests/tests_cinder_nfs.py
    create mode 100644 tox.ini
    create mode 100644 unit_tests/__init__.py
    create mode 100644 unit_tests/test_cinder_nfs_charm.py

It's important to verify that the versions match what you expect. For example,
in the above output we can see that ``tests/bundles/bionic-xena.yaml``
was generated but we'd expect ``tests/bundles/focal-xena.yaml`` so we'll need
to update that file name as well as its content.

.. code-block:: none

   mv tests/bundles/bionic-xena.yaml tests/bundles/focal-xena.yaml
   sed -i 's/bionic/focal/g' tests/bundles/focal-xena.yaml

Build the charm
~~~~~~~~~~~~~~~

The charm now needs to be built to pull down all the dependencies and
rolled into the built charm which can then be deployed.

.. code-block:: none

   cd ~/cinder-nfs
   tox -e build

Deploy the charm
~~~~~~~~~~~~~~~~

.. code-block:: none

   juju deploy --channel yoga/edge ch:cinder
   juju deploy ./cinder-nfs.charm
   juju integrate cinder:storage-backend cinder-nfs:storage-backend

:command:`juju status` will now show both charms deployed. The ``cinder``
status will show some missing relations but that's not an issue for this
demonstration.

Update Cinder configuration
---------------------------

During the initial deploy of this charm, the standard :command:`openstack.core`
default installer will install the packages specified in the class
CharmName.PACKAGES, but it will not do any other configuration.
In order to update ``cinder.conf`` in the cinder principal charm, this
NFS subordinate charm will need to access the ``storage-backend`` relation,
which will allow it to send configuration information to the
cinder principal charm for inclusion in ``cinder.conf`` on the co-located
machine.

In the generated charm, we can see that there's already the minimal code
to send data over the storage-backend relation in ``src/charm.py``:

.. code-block:: python

   def cinder_configuration(self, config):
       # Return the configuration to be set by the principal.
       volume_driver = ''
       options = [
           ('volume_driver', volume_driver)
       ]
       return options

What this is doing is sending structured data over the relation that the
cinder charm knows how to convert into native Cinder configuration. To
support the NFS backend, we'll need to modify the ``config.yaml`` file and
``cinder_configuration`` function.

.. code-block:: yaml

   options:
       nfs_host:
           default:
           description: IP or Hostname for NFS server
           type: string
       nfs_path:
           default:
           description: Path to the NFS export
           type: string

.. code-block:: python

   def cinder_configuration(self, config):
       # Return the configuration to be set by the principal.
       volume_driver = 'cinder.volume.drivers.nfs.NfsDriver'
       options = [
           ('volume_driver', volume_driver),
           ('nas_host', config['nfs_host']),
           ('nas_share_path', config['nfs_path']),
       ]
       return options

This tells the charm to send that configuration to the principal charm where
the **storage-backend** relation changes. Then repeat the steps in the
`Build the charm`_ section.

Upgrade to new charm
~~~~~~~~~~~~~~~~~~~~

To test the updates, upgrade the running charm to the one that was just built.

.. code-block:: none

   juju upgrade-charm --path ./cinder-nfs.charm

Verify the update
~~~~~~~~~~~~~~~~~

We can confirm that the update has been applied by checking for the driver in
the cinder.conf file:

.. code-block:: none

   juju exec --unit cinder/0 "grep NfsDriver /etc/cinder/cinder.conf"
   volume_driver = cinder.volume.drivers.nfs.NfsDriver

.. LINKS
.. _Openstack Cinder NFS: https://docs.openstack.org/cinder/xena/configuration/block-storage/drivers/nfs-volume-driver.html
.. _operator framework helpers: https://opendev.org/openstack/charm-ops-openstack
.. _charmcraft: https://github.com/canonical/charmcraft/
