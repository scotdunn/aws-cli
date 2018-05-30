The following command shows how to use the generate-data-key-without-plaintext command in the AWS CLI. This command saves the output in a JSON file.

The command uses the ``key-id`` parameter to specify the AWS KMS customer master key (CMK) that will be used to generate the data key. To specify the data key length, the command requires either a ``key-spec`` or ``number-of-bytes`` parameter. This example uses the ``key-spec`` parameter with a value of ``AES_256``. All data keys that AWS KMS generates are `Advanced Encryption Standard <https://en.wikipedia.org/wiki/Advanced_Encryption_Standard>`_ (AES) symmetric keys, so this value is equivalent to using the ``number-of-bytes`` parameter with a value of 32.

The ``query`` parameter selects the CiphertextBlob field that contains the encrypted data key. The ``output`` parameter formats that output as text, instead of JSON. Then, encrypted data key is converted from a base64-encoded string to binary data, and its saved in the encryptedDataKey file. You can safely store the encrypted data key with the encrypted data.

.. code::

    aws kms generate-data-key-without-plaintext --key-id 1234abcd-12ab-34cd-56ef-1234567890ab --key-spec AES_256 --query CiphertextBlob --output text  | base64 --decode > encryptedDataKey

To decrypt the encrypted data key, use the `Decrypt <decrypt.html>_` operation. The following command decrypts the encrypted data key, selects the Plaintext field from the response, base64-decodes it, and saves it in the $plaintext variable. Use the plaintext key and dispose of it as soon as possible.
    
.. code::
    
    $plaintext=$(aws kms decrypt --ciphertext-blob fileb://encryptedDataKey --query Plaintext --output text | base64 --decode)


**Example: Using an encryption context**

The following command uses an encryption context in a command to generate a data key. In AWS KMS, an `encryption context <https://docs.aws.amazon.com/kms/latest/developerguide/encryption-context.html>`_ is a collection of non-secret name-value pairs that are cryptographically bound to the encrypted data. To decrypt the encrypted data key, you need to provide the same encryption context. You can also use the encryption context to `control access to a CMK <https://docs.aws.amazon.com/kms/latest/developerguide/encryption-context.html#encryption-context-authorization>`_ and `identify the operation <https://docs.aws.amazon.com/kms/latest/developerguide/encryption-context.html#encryption-context-auditing>`_ CloudTrail logs.

In this example, the encryption context consists of two name-value pairs, ``Project=125`` and ``Purpose=Test``, separated by a comma. This example uses the shorthand syntax, but you can use JSON syntax or specify the path to a file that contains the encryption context in JSON or shorthand syntax.

If the command to decrypt this data does not include the same encryption context, the Decrypt operation fails with an ``InvalidCiphertextException`` exception.

.. code::

    aws kms generate-data-key-without-plaintext --key-id 1234abcd-12ab-34cd-56ef-1234567890ab --number-of-bytes 32 --encryption-context Project=125,Purpose=Test

If you use the JSON format at a Windows command prompt, be sure to use a backslash character (\) to escape all quotation marks inside the curly braces. For example: 

.. code::

    aws kms generate-data-key --key-id 1234abcd-12ab-34cd-56ef-1234567890ab --number-of-bytes 32 --encryption-context '{\"Project\": \"125\",\"Purpose\": \"Test\" }'