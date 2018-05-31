Example: Create an CMK alias
############################

The following command creates the ``example-alias`` alias. It associates the alias with the customer master key (CMK) that has CMK ID ``1234abcd-12ab-34cd-56ef-1234567890ab``.

Alias names must begin with ``alias/``. Do not use alias names that begin with ``alias/aws``; they are reserved for use by AWS. To associate the alias with a different CMK, use the `Decrypt <decrypt.html>_` `UpdateAlias <update-alias.html>`_ operation.

.. code::

    aws kms create-alias --alias-name alias/example-alias --target-key-id 1234abcd-12ab-34cd-56ef-1234567890ab