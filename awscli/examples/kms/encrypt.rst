The following command shows the recommended way to encrypt data with the AWS CLI.

.. code::

    aws kms encrypt --key-id 1234abcd-12ab-34cd-56ef-1234567890ab --plaintext fileb://ExamplePlaintextFile --output text --query CiphertextBlob | base64 --decode > ExampleEncryptedFile

The command does several things:

#. Uses the ``fileb://`` prefix in the value of the ``--plaintext`` parameter.

    The ``fileb://`` prefix tells the CLI to get the plaintext data from a binary file. If the file is not in the current directory, type the full path to file. For example: ``fileb:///var/tmp/ExamplePlaintextFile`` or ``fileb://C:\Temp\ExamplePlaintextFile``.

    For more information about reading AWS CLI parameter values from a file, see `Loading Parameters from a File <https://docs.aws.amazon.com/cli/latest/userguide/cli-using-param.html#cli-using-param-file>`_ in the *AWS Command Line Interface User Guide* and `Best Practices for Local File Parameters <https://blogs.aws.amazon.com/cli/post/TxLWWN1O25V1HE/Best-Practices-for-Local-File-Parameters>`_ on the AWS Command Line Tool Blog.

#. Uses the ``--output`` and ``--query`` parameters to control the command's output.

    These parameters select the CiphertextBlob field of the response, which contains the encrypted data, and returns the output as text, rather than JSON.

    For more information about controlling output, see `Controlling Command Output <https://docs.aws.amazon.com/cli/latest/userguide/controlling-output.html>`_ in the *AWS Command Line Interface User Guide*.

#. Uses the ``base64`` utility to decode the extracted output.

    When you use the CLI, the response is Base64-encoded text. You must decode this text before you can use the AWS CLI to decrypt it. This command uses the ``base64`` utility to convert the Base64-encoded text to binary data. 

#. Saves the binary ciphertext to a file.

    The final part of the command (``> ExampleEncryptedFile``) saves the binary ciphertext to a file to make decryption easier. For an example command that uses the AWS CLI to decrypt data, see the `decrypt examples <decrypt.html#examples>`_.


**Example: Using an encryption context **

The following command uses an encryption context in a command to encrypt a 'hello world' string. In AWS KMS, an `encryption context <https://docs.aws.amazon.com/kms/latest/developerguide/encryption-context.html>`_ is a collection of non-secret name-value pairs that are cryptographically bound to the encrypted data. To decrypt the data, you need to provide the same encryption context. You can also use the encryption context to control access to a CMK and identify the Encrypt operation CloudTrail logs.

In this example, the encryption context consists of two name-value pairs, ``Dept=IT`` and ``Purpose=Test``, separated by a comma. This example uses the shorthand syntax, but you can use JSON syntax or specify the path to a file that contains the encryption context in JSON or shorthand syntax.

If the command to decrypt this data does not include the same encryption context, the Decrypt operation fails with an ``InvalidCiphertextException`` exception.

.. code::

    aws kms encrypt --key-id 1234abcd-12ab-34cd-56ef-1234567890ab --plaintext 'hello world' --encryption-context Dept=IT,Purpose=Test --output text --query CiphertextBlob | base64 --decode > ExampleEncryptedMessage

If you use the JSON format at a Windows command prompt, be sure to use a backslash character (\) to escape all quotation marks inside the curly braces. For example: 

.. code::

    aws kms encrypt --key-id 1234abcd-12ab-34cd-56ef-1234567890ab --plaintext 'hello world' --encryption-context '{\"Dept\": \"IT\",\"Purpose\": \"Test\"}' --output text --query CiphertextBlob > C:\Temp\ExampleEncryptedMessage.txt
    
**Example: Using the AWS CLI to encrypt data from the Windows command prompt**

The preceding example assumes the ``base64`` utility is available, which is commonly the case on Linux and macOS. For the Windows command prompt, use ``certutil`` instead of ``base64``. This requires two commands, as shown in the following examples.

You can also use the `Invoke-KMSEncrypt <https://docs.aws.amazon.com/powershell/latest/reference/items/Invoke-KMSEncrypt.html>`_ cmdlet in the AWSPowerShell module on Windows, macOS, or Linux systems.


.. code::

    aws kms encrypt --key-id 1234abcd-12ab-34cd-56ef-1234567890ab --plaintext fileb://ExamplePlaintextFile --output text --query CiphertextBlob > C:\Temp\ExampleEncryptedFile.base64

.. code::

    certutil -decode C:\Temp\ExampleEncryptedFile.base64 C:\Temp\ExampleEncryptedFile