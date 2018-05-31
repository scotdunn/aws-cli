Example: Encrypt binary data in a file
######################################

The following command shows the recommended way to encrypt data with the AWS CLI.

.. code::

    aws kms encrypt --key-id 1234abcd-12ab-34cd-56ef-1234567890ab --plaintext fileb://ExamplePlaintextFile --output text --query CiphertextBlob | base64 --decode > ExampleEncryptedFile

About this command:

* Identifies the CMK

    The command uses the ``key-id`` parameter to specify the AWS KMS customer master key (CMK) that is used to encrypt the data. The value is a CMK ID, but you can use a CMK ARN, an alias, or an alias ARN.

* Uses the ``fileb://`` prefix in the value of the ``--plaintext`` parameter.

    The ``fileb://`` prefix tells the CLI to get the plaintext data from a binary file. If the file is not in the current directory, type the full path to file. For example: ``fileb:///var/tmp/ExamplePlaintextFile`` or ``fileb://C:\Temp\ExamplePlaintextFile``.

    For more information about reading AWS CLI parameter values from a file, see `Loading Parameters from a File <https://docs.aws.amazon.com/cli/latest/userguide/cli-using-param.html#cli-using-param-file>`_ in the *AWS Command Line Interface User Guide* and `Best Practices for Local File Parameters <https://blogs.aws.amazon.com/cli/post/TxLWWN1O25V1HE/Best-Practices-for-Local-File-Parameters>`_ on the AWS Command Line Tool Blog.

* Uses the ``--output`` and ``--query`` parameters to control the command's output.

    These parameters select the CiphertextBlob field of the response, which contains the encrypted data, and returns the output as text, rather than JSON.

    For more information about controlling output, see `Controlling Command Output <https://docs.aws.amazon.com/cli/latest/userguide/controlling-output.html>`_ in the *AWS Command Line Interface User Guide*.

* Uses the ``base64`` utility to decode the extracted output.

    When you use the CLI, the response is Base64-encoded text. You must decode this text before you can use the AWS CLI to decrypt it. This command uses the ``base64`` utility to convert the Base64-encoded text to binary data. 

* Saves the binary ciphertext to a file.

    The final part of the command (``> ExampleEncryptedFile``) saves the binary ciphertext to a file to make decryption easier. For an example command that uses the AWS CLI to decrypt data, see the `decrypt examples <decrypt.html#examples>`_.


Example: Encrypting data in Windows (cmd.exe)
#############################################

The preceding example assumes the ``base64`` utility is available, which is commonly the case on Linux and macOS. When using ``cmd.exe`` in Windows, use ``certutil`` instead of ``base64``. This requires two commands, as shown in the following examples.

You can also use the `Invoke-KMSEncrypt <https://docs.aws.amazon.com/powershell/latest/reference/items/Invoke-KMSEncrypt.html>`_ cmdlet in the AWSPowerShell module on Windows, macOS, or Linux systems.

.. code::

    aws kms encrypt --key-id 1234abcd-12ab-34cd-56ef-1234567890ab --plaintext fileb://ExamplePlaintextFile --output text --query CiphertextBlob > C:\Temp\ExampleEncryptedFile.base64

.. code::

    certutil -decode C:\Temp\ExampleEncryptedFile.base64 C:\Temp\ExampleEncryptedFile

Example: Using an encryption context
####################################

The following examples show how to use an encryption context in a command to encrypt a 'hello world' string. 

In AWS KMS, an `encryption context <https://docs.aws.amazon.com/kms/latest/developerguide/encryption-context.html>`_ is a collection of non-secret name-value pairs that are cryptographically bound to the encrypted data. To decrypt the data, you need to provide the same encryption context. Otherwise, the Decrypt operation fails with an ``InvalidCiphertextException`` exception. You can also use the encryption context to `control access to a CMK <https://docs.aws.amazon.com/kms/latest/developerguide/encryption-context.html#encryption-context-authorization>`_ and `identify the operation <https://docs.aws.amazon.com/kms/latest/developerguide/encryption-context.html#encryption-context-auditing>`_ CloudTrail logs. 

Part 1: Simple encryption context
=================================

In the following example, the encryption context is ``Dept=IT``. Each command demonstrates a different command syntax, but they are all equivalent.

* Shorthand syntax:
.. code::

    aws kms encrypt --encryption-context Dept=IT --key-id 1234abcd-12ab-34cd-56ef-1234567890ab --plaintext 'hello world' --output text --query CiphertextBlob | base64 --decode > ExampleEncryptedMessage

* JSON
.. code::

    aws kms encrypt --encryption-context '{"Dept": "IT"}' --key-id 1234abcd-12ab-34cd-56ef-1234567890ab --plaintext 'hello world' --output text --query CiphertextBlob | base64 --decode > ExampleEncryptedMessage

* File containing JSON or shorthand syntax
.. code::

    aws kms encrypt --encryption-context file://encryptionContext --key-id 1234abcd-12ab-34cd-56ef-1234567890ab --plaintext 'hello world' --output text --query CiphertextBlob | base64 --decode > ExampleEncryptedMessage
    
If you use the JSON format in a Windows command prompt (``cmd.exe``), use a backslash character (\) to escape all quotation marks inside the curly braces, as shown in the following example.
.. code::

    aws kms encrypt --encryption-context '{\"Dept\": \"IT\"}' --key-id 1234abcd-12ab-34cd-56ef-1234567890ab --plaintext 'hello world' --output text --query CiphertextBlob > C:\Temp\ExampleEncryptedMessage.txt


Part 1: Multi-value encryption context
======================================

The encryption context can include multiple name-value pairs separated by a comma. In this example, the encryption context consists of two name-value pairs, ``Dept=IT`` and ``Purpose=Test``. Each command demonstrates a different command syntax, but they are all equivalent.

* Shorthand syntax:
.. code::

    aws kms encrypt --encryption-context Dept=IT,Purpose=Test --key-id 1234abcd-12ab-34cd-56ef-1234567890ab --plaintext 'hello world' --output text --query CiphertextBlob | base64 --decode > ExampleEncryptedMessage

* JSON
.. code::

    aws kms encrypt --encryption-context '{"Dept": "IT","Purpose": "Test"}' --key-id 1234abcd-12ab-34cd-56ef-1234567890ab --plaintext 'hello world' --output text --query CiphertextBlob | base64 --decode > ExampleEncryptedMessage

* File containing JSON or shorthand syntax
.. code::

    aws kms encrypt --encryption-context file://encryptionContext --key-id 1234abcd-12ab-34cd-56ef-1234567890ab --plaintext 'hello world' --output text --query CiphertextBlob | base64 --decode > ExampleEncryptedMessage
    
If you use the JSON format in a Windows command prompt (``cmd.exe``), be sure to use a backslash character (\) to escape all quotation marks inside the curly braces. For example: 
.. code::

    aws kms encrypt --encryption-context '{\"Dept\": \"IT\",\"Purpose\": \"Test\"}' --key-id 1234abcd-12ab-34cd-56ef-1234567890ab --plaintext 'hello world' --output text --query CiphertextBlob > C:\Temp\ExampleEncryptedMessage.txt
    
