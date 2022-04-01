=======================================
Emulated Trusted Platform Module (vTPM)
=======================================

Overview
--------

`Trusted Platform Modules`_ can be used to enhance computer security and
privacy. TPM is even required by some Operating Systems.

To support TPM devices within guest instances, OpenStack Nova integrates with
software-based emulated TPM devices for QEMU and KVM guest instances. The
secrets stored within the emulated devices are encrypted using Barbican
secrets. The devices are then provided via the :command:`swtpm` software
package.

Pre-requisites
--------------

The following requirements must be met in order to enable vTPM support in the
nova-compute charm:

* OpenStack Wallaby or newer
* Barbican Key Manager service must be deployed and configured
* swtpm libraries must be available for installation

If you are using an apt mirror, make sure it contains the ``swtpm``,
``swtpm-tools``, and ``libtpms0`` packages.

.. note::

   The swtpm, swtpm-tools, and libtpms libraries are available in Ubuntu 22.04
   LTS (Jammy) release. It is expected that they will be backported to the
   Ubuntu 20.04 LTS (Focal) archives. Until this is done, the OpenStack Charms
   team is providing a Personal Package Archive (PPA) with the necessary
   packages for Focal.

Deployment
----------

TPM support is enabled on all compute nodes by using the nova-compute charm's
``enable-vtpm`` configuration option.

In this example, support is enabled on Focal-based nodes via a PPA. The
following YAML excerpt contains the configuration:

.. code-block:: yaml

   nova-compute:
     enable-vtpm: True
     extra-repositories: ppa:openstack-charmers/swtpm

Nova will use the credentials for service discovery from Keystone in order to
determine the Barbican endpoint to use.

Once vTPM support has been enabled in the compute nodes, verify that the
compute nodes are registering the TPM traits within the Placement service:

.. code-block:: none

   COMPUTE_UUID=$(openstack resource provider list --name $HOST -f value -c uuid)
   openstack resource provider trait list $COMPUTE_UUID | grep SECURITY_TPM
   | COMPUTE_SECURITY_TPM_1_2 |
   | COMPUTE_SECURITY_TPM_2_0 |

OpenStack configuration
-----------------------

TPM support is added to a VM by means of an OpenStack flavor. This will specify
the TPM version and model for the vTPM device to emulate.

There are two versions to choose from (1.2 and 2.0) as well as two model types
(tpm-tis and tpm-crb).

.. note::

   The default model is 'tpm-tis'.

   The tpm-crb model is only compatible with TPM version 2.0

The following example configures an existing flavor to use TPM 2.0 with the CRB
model (optionally create a new flavor):

.. code-block:: none

   openstack flavor set <flavor-name> \
     --property hw:tpm_version=2.0 \
     --property hw:tpm_model=tpm-crb

The image used to create a TPM-supported VM must be configured to use UEFI
firmware. This is done by setting the ``hw_firmware_type`` property to
``uefi``.

The following example configures an existing image to use UEFI (optionally
import a new image):

.. code-block:: none

   openstack image set <image-name-or-uuid> --property hw_firmware_type=uefi

References
----------

More information related to the usage of vTPM can be found in the upstream
OpenStack documentation:

* `Emulated Trusted Platform Module`_ (Nova)
* `Extra Specs`_ (Nova)
* `Secure Boot`_ (Nova)
* `Useful image properties`_ (Glance)

.. LINKS
.. _Emulated Trusted Platform Module: https://docs.openstack.org/nova/latest/admin/emulated-tpm.html
.. _Extra Specs: https://docs.openstack.org/nova/latest/configuration/extra-specs.html
.. _Secure Boot: https://docs.openstack.org/nova/latest/admin/secure-boot.html
.. _Trusted Platform Modules: https://en.wikipedia.org/wiki/Trusted_Platform_Module
.. _Useful image properties: https://docs.openstack.org/glance/latest/admin/useful-image-properties.html

