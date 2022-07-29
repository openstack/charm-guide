.. role:: raw-html(raw)
   :format: html

========
Keystone
========

Security compliance
-------------------

The keystone charm's ``password-security-compliance`` configuration option sets
the ``[security_compliance]`` section of Keystone's configuration file. The
value of this option is a YAML dictionary that includes support for the
following keys (value formats and units are also included).

.. code-block:: none

   change_password_upon_first_use: <boolean>
   disable_user_account_days_inactive: <int> (days)
   lockout_duration: <int> (seconds)
   lockout_failure_attempts: <int>
   minimum_password_age: <int> (days)
   password_expires_days: <int> (days)
   password_regex: <string>
   password_regex_description: <string>
   unique_last_password_count: <int>

.. important::

   The upstream document `Security compliance and PCI-DSS`_ should be consulted
   before setting any of these options.

The configuration is typically contained within a file, say ``config.yaml``.
For example:

.. code-block:: yaml

   password-security-compliance:
      change_password_upon_first_use: True
      lockout_duration: 1800
      lockout_failure_attempts: 3
      ...

It is applied in the usual way:

.. code-block:: none

   juju config keystone --file config.yaml

The charm will protect service accounts (accounts requested by other units that
are in the service domain) against being forced to change their password.
Operators should also ensure that any other accounts are protected as per the
above referenced note.

Operators should also ensure that any non-service accounts are protected as per
the upstream document.

The charm will enter a blocked state if the value of charm option
``password-security-compliance`` is not in valid YAML format and/or the
individual service options do not conform to the proper value formats.

.. _keystone_tokens:

Token support
-------------

Over time OpenStack has come to support two Keystone token formats: UUID and
Fernet.

Fernet tokens were added to address the issue of size observed with PKI and
PKIZ tokens in addition to continuing the Keystone behaviour of persisting
tokens to a common database cluster (like UUID tokens). For more information
see this `Fernet FAQ`_.

.. important::

   Starting with OpenStack Rocky, only the Fernet format for authentication
   tokens is supported. This is documented on the
   :doc:`cdg:upgrade-queens-to-rocky` page in the Deploy Guide.

Fernet keys
-----------

Theory of operation
~~~~~~~~~~~~~~~~~~~

Keystone generates Fernet keys, which in turn are used to generate/encrypt
and decode/decrypt Fernet tokens.

There are three key types, and each is associated with a naming scheme that is
applied to the directories in which they are found. Directory names are based
on integers:

* primary key
   * integer: the highest one
   * generates tokens
   * number of keys: one

* staged key
   * integer: '0'
   * will become the next primary key
   * can decode tokens
   * number of keys: one

* secondary key
   * integer: any other number
   * was previously a primary key
   * can decode tokens
   * number of keys: one or more

Each key type must be present at all times. This means Keystone uses a minimum
of three keys.

Key rotation
~~~~~~~~~~~~

Key rotation refers to the process by which two keys assume a different type,
one key gets created, and typically one key gets removed:

#. staged :raw-html:`&rarr;` primary
#. primary :raw-html:`&rarr;` secondary
#. the staged is created
#. a secondary is removed

A key will get removed if the total number of keys surpasses the specified
maximum allowed (more on this later).

This process takes place on the master keystone unit and takes into account
three aspects:

* rotation frequency
* token expiration
* maximum number of active keys

These are related according to this formula:

.. math::

   max\_active\_keys = \frac{ token\_expiration }{ rotation\_frequency } + 2

In the keystone charm, token expiration and the maximum number of active keys
are specified, respectively, with the ``token-expiration`` and the
``fernet-max-active-keys`` configuration options.

For example, given that an administrator desires a token expiration of 1 hour
(3600 seconds) and a rotation frequency of 15 minutes (900 seconds), the maximum
number of active keys must be six:

.. math::

   \begin{eqnarray}
   max\_active\_keys &=& \frac{ 3600 }{ 900 } + 2\\
   &=& 4+2\\
   &=& 6\\
   \end{eqnarray}

The above two options can then be set accordingly:

.. code-block:: yaml

   token-expiration: 3600
   fernet-max-active-keys: 6

Rotation frequency
^^^^^^^^^^^^^^^^^^

From the point of view of rotation frequency:

.. math::

   rotation\_frequency = \frac{ token\_expiration }{ max\_active\_keys - 2 }

Since the denominator must lead to a positive real number for rotation
frequency the value of ``fernet-max-active-keys`` must be at least three, and
this constraint is enforced by the charm.

To increase rotation frequency either decrease ``fernet-max-active-keys`` or
increase ``token-expiration``. To decrease rotation frequency, do the opposite.

The most notable effect of increasing rotation frequency is the reduction in
key lifetime (secondary keys get removed more often).

Default values
^^^^^^^^^^^^^^

These are the default values for these keystone charm options and the resulting
default rotation frequency:

* ``token-expiration``: 3600 sec (1 hour)
* ``fernet-max-active-keys``: 3
* ``rotation frequency``: 3600 sec (1 hour)

Token validation breakage
~~~~~~~~~~~~~~~~~~~~~~~~~

Token validation breakage is a situation in which a decoding key is no longer
available to validate an unexpired token. This can be caused by a rotation
frequency that has been set too high (a very short key lifetime) or by keys
failing to synchronise (from the master keystone unit to the other units) prior
to the succeeding rotation. Incremental changes to rotation frequency is
therefore advised.

.. LINKS
.. _Security compliance and PCI-DSS: https://docs.openstack.org/keystone/latest/admin/configuration.html#security-compliance-and-pci-dss
.. _Fernet FAQ: https://docs.openstack.org/keystone/pike/admin/identity-fernet-token-faq.html
