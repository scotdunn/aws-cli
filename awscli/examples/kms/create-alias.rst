Example: Create an CMK alias
############################

The following command creates the ``example-alias`` alias. It associates the alias with the customer master key (CMK) that has CMK ID ``1234abcd-12ab-34cd-56ef-1234567890ab``.

When specifying an alias, type ``alias/`` before the alias name. For example, ``alias/myAlias``. Alias names cannot begin with ``aws``; that name is reserved for AWS aliases. To associate the alias with a different CMK, use the `UpdateAlias <update-alias.html>`_ operation.

.. code::

    aws kms create-alias --alias-name alias/exampleAlias --target-key-id 1234abcd-12ab-34cd-56ef-1234567890ab
