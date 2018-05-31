Example: Generate a data key without plaintext
##############################################

This example shows how to use the ``generate-data-key-without-plaintext`` command in the AWS CLI. The output contains a copy of the data key encrypted under the CMK. 

Step 1: Generate the data key
=============================

The following command gets the encrypted data key from the response, Base64-decodes it, and saves it in a file. Then, it shows how to use the `Decrypt <decrypt.html>`_ operation to decrypt the encrypted data key so you can use it to encrypt or decrypt data outside of AWS KMS.

.. code::

    aws kms generate-data-key-without-plaintext --key-id 1234abcd-12ab-34cd-56ef-1234567890ab --key-spec AES_256 --query CiphertextBlob --output text  | base64 --decode > encryptedDataKey

About this command:

* Identifies the CMK.

    The command uses the ``key-id`` parameter to specify the AWS KMS customer master key (CMK) that will be used to generate the data key. 
    
* Specifies the key length.

    To specify the data key length, the command requires either a ``key-spec`` or ``number-of-bytes`` parameter. This example uses the ``key-spec`` parameter with a value of ``AES_256``. All data keys that AWS KMS generates are `Advanced Encryption Standard <https://en.wikipedia.org/wiki/Advanced_Encryption_Standard>`_ (AES) symmetric keys, so this value is equivalent to using the ``number-of-bytes`` parameter with a value of 32.

* Gets the encrypted data key.

    The ``query`` parameter selects the CiphertextBlob field that contains the encrypted data key from the response. The ``output`` parameter formats that output as text, instead of JSON. 

* Base64-decodes the output.

    All fields in an AWS CLI response are Base64 encoded string. Before saving the encrypted data key in a file, this command uses the ``base64`` utility to decode it. In Windows, use ``certutil`` or a PowerShell cmdlet.

* Saves the output.

    The command saves the output in the ``encryptedDataKey`` file.

Step 2: Decrypt the data key
=============================
    
To decrypt the encrypted data key, use the `Decrypt <decrypt.html>`_ operation. The following command decrypts the encrypted data key, selects the ``Plaintext`` field from the response, base64-decodes it, and saves it in the ``$plaintext`` variable. After using the plaintext key, delete it as soon as possible.
    
.. code::
    
    $plaintext=$(aws kms decrypt --ciphertext-blob fileb://encryptedDataKey --query Plaintext --output text | base64 --decode)


Example: Using an encryption context
####################################

In AWS KMS, an `encryption context <https://docs.aws.amazon.com/kms/latest/developerguide/encryption-context.html>`_ is a collection of non-secret name-value pairs that are cryptographically bound to the encrypted data. To decrypt the encrypted data, you need to provide the same encryption context. Otherwise, the Decrypt operation fails with an ``InvalidCiphertextException`` exception. You can also use the encryption context to `control access to a CMK <https://docs.aws.amazon.com/kms/latest/developerguide/encryption-context.html#encryption-context-authorization>`_ and `identify the operation <https://docs.aws.amazon.com/kms/latest/developerguide/encryption-context.html#encryption-context-auditing>`_ CloudTrail logs.

In this example, the encryption context is one name-value pair, ``Project=125``. This example uses the shorthand syntax, but you can use JSON syntax or specify the path to a file that contains the encryption context in JSON or shorthand syntax. For a more detailed example that shows all formats, see the `Example: Using an encryption context <https://github.com/juneb/aws-cli/blob/kms-examples/awscli/examples/kms/encrypt.rst#example-using-an-encryption-context>`_. You do not need to use the same syntax in the ``encrypt`` and ``decrypt`` commands.

.. code::

    aws kms generate-data-key-without-plaintext --encryption-context Project=125 --key-id 1234abcd-12ab-34cd-56ef-1234567890ab --number-of-bytes 32

If you use the JSON format at a Windows command prompt (``cmd.exe``), be sure to use a backslash character (\\) to escape all quotation marks inside the curly braces. For example: 

.. code::

    aws kms generate-data-key --key-id 1234abcd-12ab-34cd-56ef-1234567890ab --number-of-bytes 32 --encryption-context '{\"Project\": \"125\"}'