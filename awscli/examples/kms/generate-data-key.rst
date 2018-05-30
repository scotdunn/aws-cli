The following command shows how to use the generate-data-key command in the AWS CLI. It saves the response in the $dataKeyResponse variable.

The command uses the ``key-id`` parameter to specify the AWS KMS customer master key (CMK) that will be used to generate and encrypt the data key. To specify the data key length, the command requires either a ``key-spec`` or ``number-of-bytes`` parameter. This example uses the ``key-spec`` parameter with a value of ``AES_256``. All data keys that AWS KMS generates are `Advanced Encryption Standard <https://en.wikipedia.org/wiki/Advanced_Encryption_Standard>`_ (AES) symmetric keys, so this value is equivalent to using the ``number-of-bytes`` parameter with a value of 32.

.. code::

    dataKeyResponse=$(aws kms generate-data-key --key-id 1234abcd-12ab-34cd-56ef-1234567890ab --key-spec AES_256)

The response includes a plaintext data key (``Plaintext``), a copy of that key encrypted under the CMK (``CiphertextBlob``), and the Amazon Resource Name (ARN) of the CMK that generated and encrypted the data keys (``KeyId``), all of which are represented by Base64-encoded strings. You can use the ``KeyId`` to verify that the operation used the intended CMK. Then, use the plaintext key to encrypt data outside of KMS, and dispose of it. You can safely store the encrypted data key with the encrypted data.

.. code::
    echo $dataKeyResponse

    {
        "Plaintext": "CZHRuyc8yXDiGBjOAyln9Atlyy+0XV4js6E0Mr+wKPs=",
        "CiphertextBlob": "AQIDAHialP3mtJTkmLqUHBGW6bsnZDiaXUc+nSYIDAu/H90RxwHUrZJEocwaCN3LgTbL1hnWAAAAfjB8BgkqhkiG9w0BBwagbzBtAgEAMGgGCSqGSIb3DQEHATAeBglghkgBZQMEAS4wEQQMQfk0OjmPhnY89mfWAgEQgDvvS+CkDjT9C7VgZ058KbKMRjt9h86sJwoKRTY9lRh6TH9YLCvVhB5XvoJmX5uUNW2CI0w0gkgyLocddg==",
        "KeyId": "arn:aws:kms:us-west-2:111122223333:key/1234abcd-12ab-34cd-56ef-1234567890ab"
    }


Next, save the plaintext data key in a variable so you can use it to encrypt data and then dispose of it. The following command gets the plaintext key from the $dataKeyResponse variable, Base64-decodes it, and saves it in the $plaintext variable. To get the plaintext key, this example uses the jq utility with the -r command that excludes quotation marks. JQ is available for Windows, macOS, and Linux systems. For PowerShell, use the `ConvertFrom-JSON <https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/convertfrom-json>`_ cmdlet. 
    
.. code::

    plaintext=$(jq -r '.Plaintext'<<< $dataKeyResponse | base64 -d)

Now, you can save the encrypted data key in a file. The following command gets the encrypted data key from the $dataKeyResponse variable, base64-decodes it, and saves it in the encryptedDataKey file. To use the encrypted data key, use the `Decrypt <decrypt.html>`_ command to decrypt it.
    
.. code::
    
    jq -r '.CiphertextBlob'<<< $dataKeyResponse | base64 -d > encryptedDataKey

When you're ready to decrypt the data that you encrypted with the plaintext key, you need decrypt the encrypted data key, as shown in the following command. 

This command uses the `Decrypt <decrypt.html>_` operation to decrypt the encrypted data key. The value of the ciphertext-blob parameter is the file that contains the encrypted data key as binary data. The command selects the Plaintext field of the response, returns it as text, and Base64-decodes it before saving it in the $decryptedDataKey variable.

.. code::
    
    $decryptedDataKey=$(aws kms decrypt --ciphertext-blob fileb://encryptedDataKey --query Plaintext --output text | base64 --decode)

**Example: Using an encryption context**

The following command uses an encryption context in a command to generate a data key. In AWS KMS, an `encryption context <https://docs.aws.amazon.com/kms/latest/developerguide/encryption-context.html>`_ is a collection of non-secret name-value pairs that are cryptographically bound to the encrypted data. To decrypt the encrypted data, you need to provide the same encryption context. You can also use the encryption context to `control access to a CMK <https://docs.aws.amazon.com/kms/latest/developerguide/encryption-context.html#encryption-context-authorization>`_ and `identify the operation <https://docs.aws.amazon.com/kms/latest/developerguide/encryption-context.html#encryption-context-auditing>`_ CloudTrail logs.

In this example, the encryption context consists of two name-value pairs, ``Project=125`` and ``Purpose=Test``, separated by a comma. This example uses the shorthand syntax, but you can use JSON syntax or specify the path to a file that contains the encryption context in JSON or shorthand syntax.

If the command to decrypt this data does not include the same encryption context, the Decrypt operation fails with an ``InvalidCiphertextException`` exception.

.. code::

    aws kms generate-data-key --key-id 1234abcd-12ab-34cd-56ef-1234567890ab --number-of-bytes 32 --encryption-context Project=125,Purpose=Test

If you use the JSON format at a Windows command prompt, be sure to use a backslash character (\) to escape all quotation marks inside the curly braces. For example: 

.. code::

    aws kms generate-data-key --key-id 1234abcd-12ab-34cd-56ef-1234567890ab --number-of-bytes 32 --encryption-context '{\"Project\": \"125\",\"Purpose\": \"Test\" }'