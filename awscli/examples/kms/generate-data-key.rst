Example: Generate a data key
############################

This example shows how to use the ``generate-data-key`` command from the AWS CLI. The output contains a plaintext data key and a copy of the data key encrypted under the CMK. In this example, the commands save the plaintext data key in a variable and write the encrypted data key to a file. Because the values are base64-encoded, the commands base64-decode the values before saving them.

Step 1: Generate the data key
=============================
The following command uses an AWS KMS CMK to generate a data key.

.. code::

    dataKeyResponse=$(aws kms generate-data-key --key-id 1234abcd-12ab-34cd-56ef-1234567890ab --key-spec AES_256)

About this command:
    
* Identifies the CMK.

    The command uses the ``key-id`` parameter to specify the AWS KMS customer master key (CMK) that generates and encrypts the data key. The value is a CMK ID, but you can use a CMK ARN, an alias, or an alias ARN.

* Specifies the data key length.

    The command requires either a ``key-spec`` or ``number-of-bytes`` parameter. This example uses the ``key-spec`` parameter with a value of ``AES_256``. All data keys that AWS KMS generates are `Advanced Encryption Standard <https://en.wikipedia.org/wiki/Advanced_Encryption_Standard>`_ (AES) symmetric keys, so this value is equivalent to using the ``number-of-bytes`` parameter with a value of 32.

* Saves the response

    This command saves the response in the ``$dataKeyResponse`` variable. Because the response includes a plaintext data key, we don't write it to a file.

Step 2: Examine the output
==========================
The following command displays the command response in the ``$dataKeyResponse`` variable.

The response includes a plaintext data key (``Plaintext``), a copy of that key encrypted under the CMK (``CiphertextBlob``), and the Amazon Resource Name (ARN) of the CMK that generated and encrypted the data keys (``KeyId``), all of which are represented by base64-encoded strings. 

You can use the ``KeyId`` to verify that the operation used the intended CMK. Then, use the plaintext key to encrypt data outside of KMS, and dispose of it. You can safely store the encrypted data key with the encrypted data.

.. code::

    echo $dataKeyResponse

    {
        "Plaintext": "CZHRuyc8yXDiGBjOAyln9Atlyy+0XV4js6E0Mr+wKPs=",
        "CiphertextBlob": "AQIDAHialP3mtJTkmLqUHBGW6bsnZDiaXUc+nSYIDAu/H90RxwHUrZJEocwaCN3LgTbL1hnWAAAAfjB8BgkqhkiG9w0BBwagbzBtAgEAMGgGCSqGSIb3DQEHATAeBglghkgBZQMEAS4wEQQMQfk0OjmPhnY89mfWAgEQgDvvS+CkDjT9C7VgZ058KbKMRjt9h86sJwoKRTY9lRh6TH9YLCvVhB5XvoJmX5uUNW2CI0w0gkgyLocddg==",
        "KeyId": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab"
    }


Step 3: Get the plaintext data key
==================================
The following command saves the plaintext data key in a variable so you can use it to encrypt data and then delete it.

The command gets the plaintext key from the ``$dataKeyResponse`` variable, base64-decodes it, and saves it in the ``$plaintext`` variable. 

To get the plaintext key, this command uses the **JQ** utility with the ``-r`` parameter that excludes quotation marks. JQ is available for Windows, macOS, and Linux systems. In PowerShell, use the `ConvertFrom-JSON <https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/convertfrom-json>`_ cmdlet.     
    
.. code::

    plaintext=$(jq -r '.Plaintext'<<< $dataKeyResponse | base64 -d)


Step 4: Save the encrypted data key
===================================
The following command gets the encrypted data key from the ``$dataKeyResponse`` variable, base64-decodes it, and saves it in the ``encryptedDataKey`` file. 
    
.. code::

    jq -r '.CiphertextBlob'<<< $dataKeyResponse | base64 -d > encryptedDataKey


Step 5: Decrypt the encrypted data key
======================================
When you're ready to decrypt the data that you encrypted with the plaintext key, you need decrypt the encrypted data key, as shown in the following command. 

.. code::

    decryptedDataKey=$(aws kms decrypt --ciphertext-blob fileb://encryptedDataKey --query Plaintext --output text | base64 --decode)

This command uses the `Decrypt <decrypt.html>`_ operation to decrypt the encrypted data key. The value of the ``ciphertext-blob`` parameter is the file that contains the encrypted data key as binary data. To get the plaintext data key from the response, the command selects the ``Plaintext`` field of the response, returns it as text, and base64-decodes it before assigning it to the ``$decryptedDataKey`` variable.

After using the plaintext key, delete it as soon as possible.

Example: Using an encryption context
####################################

The following command uses an encryption context in a command to generate a data key. 

In AWS KMS, an `encryption context <https://docs.aws.amazon.com/kms/latest/developerguide/encryption-context.html>`_ is a collection of nonsecret name-value pairs that are cryptographically bound to the encrypted data. To decrypt the encrypted data, you need to provide the same encryption context. Otherwise, the decrypt operation fails with an ``InvalidCiphertextException`` exception. You can also use the encryption context to `control access to a CMK <https://docs.aws.amazon.com/kms/latest/developerguide/encryption-context.html#encryption-context-authorization>`_ and `identify the operation <https://docs.aws.amazon.com/kms/latest/developerguide/encryption-context.html#encryption-context-auditing>`_ in CloudTrail logs.

In this example, the encryption context consists of two name-value pairs, ``Project=125`` and ``Purpose=Test``, separated by a comma. This example uses the shorthand syntax, but you can use JSON syntax or specify the path to a file that contains the encryption context in JSON or shorthand syntax. For a more detailed example that shows all formats, see the `Example: Using an encryption context <https://github.com/juneb/aws-cli/blob/kms-examples/awscli/examples/kms/encrypt.rst#example-using-an-encryption-context>`_. You do not need to use the same syntax in the ``encrypt`` and ``decrypt`` commands.

.. code::

    aws kms generate-data-key --key-id 1234abcd-12ab-34cd-56ef-1234567890ab --number-of-bytes 32 --encryption-context Project=125,Purpose=Test

If you use the JSON format at a Windows command prompt (``cmd.exe``), use a backslash character (\\) to escape all quotation marks inside the curly braces, as shown in the following command.

.. code::

    aws kms generate-data-key --key-id 1234abcd-12ab-34cd-56ef-1234567890ab --number-of-bytes 32 --encryption-context "{\"Project\": \"125\",\"Purpose\": \"Test\" }"
