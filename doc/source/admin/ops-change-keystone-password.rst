:orphan:

==================================
Change the Keystone admin password
==================================

Preamble
--------

There are valid use cases for resetting the Keystone administrator password on
a running cloud. For example, the password may have been unintentionally
exposed to a third-party during a troubleshooting session (e.g. directly on
screen, remote screen-sharing, viewing of log files, etc.).

.. warning::

   This procedure will cause downtime for Keystone, the cloud's central
   authentication service. Many core services will therefore be impacted. Plan
   for a short maintenance window (~15 minutes).

   It is recommended to first test this procedure on a staging cloud.

Procedure
---------

Confirm the admin user context
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Ensure that the current user is user 'admin':

.. code-block:: none

   env | grep OS_USERNAME
   OS_USERNAME=admin

If it's not, source the appropriate cloud admin init file (e.g. ``openrc`` or
``novarc``).

Obtain the current password
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Obtain the current password with:

.. code-block:: none

   juju run --unit keystone/leader leader-get admin_passwd

Change the password
~~~~~~~~~~~~~~~~~~~

Generate a 16-character password string with the :command:`pwgen` utility:

.. code-block:: none

   PASSWD=$(pwgen -s 16 1)

Change the password with the below command. When prompted, enter the current
password and then the new password (i.e. the output to ``echo $PASSWD``).

.. caution::

   Once the next command completes successfully the cloud will no longer be
   able to authenticate requests by the OpenStack CLI clients or the cloud's
   core services (i.e. Cinder, Glance, Neutron, Compute, Nova Cloud
   Controller).

.. code-block:: none

   openstack user password set
   Current Password: ****************
   New Password: ****************
   Repeat New Password: ****************

The entered data will not echo back to the screen.

.. note::

   Command options ``--original-password`` and ``--password`` are available but
   can leak sensitive information to the system logs.

Inform the keystone charm
~~~~~~~~~~~~~~~~~~~~~~~~~

Inform the keystone charm of the new password:

.. code-block:: none

   juju run -u keystone/leader -- leader-set 'admin_passwd=$PASSWD'

Verification
~~~~~~~~~~~~

Verify the resumption of normal cloud operations by running a routine battery
of tests. The creation of a VM is a good choice.

Update any user-facing tools
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Any cloud init files (e.g. ``novarc``) that are hardcoded with the old admin
password should be updated to guarantee continued administrative access to the
cloud by admin-level operators.

Refresh any browser-cached passwords or password-management plugins (e.g.
Bitwarden, LastPass) to ensure successful cloud dashboard (Horizon) logins.
