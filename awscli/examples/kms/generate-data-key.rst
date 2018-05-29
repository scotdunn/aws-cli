The following command shows how to use the generate-data-key command in the AWS CLI. This command saves the output in a JSON file.

The command uses the ``key-id`` parameter to specify the AWS KMS customer master key (CMK) that will be used to generate the data key. To specify the data key length, the command requires either a ``key-spec`` or ``number-of-bytes`` parameter. This example uses the ``key-spec`` parameter with a value of ``AES_256``. All data keys that AWS KMS generates are `Advanced Encryption Standard <https://en.wikipedia.org/wiki/Advanced_Encryption_Standard>_` (AES) symmetric keys, so this value is equivalent to using the ``number-of-bytes`` parameter with a value of 32.

.. code::

    aws kms generate-data-key --key-id 1234abcd-12ab-34cd-56ef-1234567890ab --key-spec AES_256 > dataKeyResponse.json

The response includes a plaintext data key (``Plaintext``), a copy of that key encrypted under the CMK (``CiphertextBlob``), and the Amazon Resource Name (ARN) of the CMK that generated and encrypted the data keys (``KeyId``). You can use the ``KeyId`` to verify that the operation used the intended CMK. Then, use the plaintext key to encrypt data outside of KMS, and dispose of it. You can safely store the encrypted data key with the encrypted data.

.. code::

    {
        "Plaintext": "CZHRuyc8yXDiGBjOAyln9Atlyy+0XV4js6E0Mr+wKPs=",
        "CiphertextBlob": "AQIDAHialP3mtJTkmLqUHBGW6bsnZDiaXUc+nSYIDAu/H90RxwHUrZJEocwaCN3LgTbL1hnWAAAAfjB8BgkqhkiG9w0BBwagbzBtAgEAMGgGCSqGSIb3DQEHATAeBglghkgBZQMEAS4wEQQMQfk0OjmPhnY89mfWAgEQgDvvS+CkDjT9C7VgZ058KbKMRjt9h86sJwoKRTY9lRh6TH9YLCvVhB5XvoJmX5uUNW2CI0w0gkgyLocddg==",
        "KeyId": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab"
    }


The following command gets the plaintext key from the JSON file, Base64-decodes it, and saves it in the encryptedDataKey file. To get the plaintext key, use the jq utility with the -r command that excludes quotation marks. JQ is available for Windows, macOS, and Linux systems. For PowerShell, use the `ConvertFrom-JSON <https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/convertfrom-json>_` cmdlet. 
    
.. code::

    plaintext=$(jq -r '.Plaintext' dataKeyResponse.js | base64 --decode)

The following command gets the encrypted data key, base64-decodes it, and saves it in the encryptedDataKey file. To use the encrypted data key, use the `Decrypt <decrypt.html>`_ command to decrypt it.
    
.. code::
    
    jq -r '.CiphertextBlob' dataKeyResponse.js | base64 --decode > encryptedDataKey


**Example: Using an encryption context **

The following command uses an encryption context in a command to generate a data key. In AWS KMS, an `encryption context <https://docs.aws.amazon.com/kms/latest/developerguide/encryption-context.html>`_ is a collection of non-secret name-value pairs that are cryptographically bound to the encrypted data. To decrypt the encrypted data key, you need to provide the same encryption context. You can also use the encryption context to control access to a CMK and identify the Encrypt operation CloudTrail logs.

In this example, the encryption context consists of two name-value pairs, ``Project=125`` and ``Purpose=Test``, separated by a comma. This example uses the shorthand syntax, but you can use JSON syntax or specify the path to a file that contains the encryption context in JSON or shorthand syntax.

If the command to decrypt this data does not include the same encryption context, the Decrypt operation fails with an ``InvalidCiphertextException`` exception.

.. code::

    aws kms generate-data-key --key-id 1234abcd-12ab-34cd-56ef-1234567890ab --number-of-bytes 32 --encryption-context Project=125,Purpose=Test > dataKeyResponse.json

If you use the JSON format at a Windows command prompt, be sure to use a backslash character (\) to escape all quotation marks inside the curly braces. For example: 

.. code::

    aws kms generate-data-key --key-id 1234abcd-12ab-34cd-56ef-1234567890ab --number-of-bytes 32 --encryption-context '{\"Project\": \"125\",\"Purpose\": \"Test\" }' > dataKeyResponse.json