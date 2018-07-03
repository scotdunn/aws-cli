Example: Encrypt binary data in a file
######################################

The following command shows the recommended way to encrypt data from the AWS CLI.

.. code::

    aws kms encrypt --key-id 1234abcd-12ab-34cd-56ef-1234567890ab --plaintext fileb://ExamplePlaintextFile --output text --query CiphertextBlob | base64 --decode > ExampleEncryptedFile

About this command:

* Identifies the CMK.

    The command uses the ``key-id`` parameter to specify the AWS KMS customer master key (CMK) that is used to encrypt the data. The value is a CMK ID, but you can use a CMK ARN, an alias, or an alias ARN.

* Gets the plaintext data from a file.

    The value of the ``plaintext`` parameter is a file path with the ``fileb://`` prefix. The ``b`` in the prefix tells the CLI that the files contains binary data.

    For more information about reading AWS CLI parameter values from a file, see `Loading Parameters from a File <https://docs.aws.amazon.com/cli/latest/userguide/cli-using-param.html#cli-using-param-file>`_ in the *AWS Command Line Interface User Guide* and `Best Practices for Local File Parameters <https://blogs.aws.amazon.com/cli/post/TxLWWN1O25V1HE/Best-Practices-for-Local-File-Parameters>`_ on the AWS Command Line Tool Blog.

* Selects the encrypted data from the response.

    The ``query`` parameter selects the ``CiphertextBlob`` field, which contains the encrypted data, from the response. The ``output`` parameter returns the output as text rather than JSON.

    For more information about controlling output, see `Controlling Command Output <https://docs.aws.amazon.com/cli/latest/userguide/controlling-output.html>`_ in the *AWS Command Line Interface User Guide*.

* Decodes the encrypted data.

    All fields in an AWS CLI response contain base64-encoded text. This command uses the ``base64`` utility to decode the encrypted data before saving it in a file.

* Saves the binary ciphertext to a file.

    The final part of the command (``> ExampleEncryptedFile``) saves the binary ciphertext to a file. For an example command that uses the AWS CLI to decrypt data, see the `decrypt examples <decrypt.html#examples>`_.


Example: Encrypting data in Windows (cmd.exe)
#############################################

The preceding example uses the ``base64`` utility, which is commonly available in Linux and macOS shells. To base64-encode and decode data in ``cmd.exe`` on Windows, use ``certutil``, as shown in the following commands. 

.. code::

    aws kms encrypt --key-id 1234abcd-12ab-34cd-56ef-1234567890ab --plaintext fileb://ExamplePlaintextFile --output text --query CiphertextBlob > C:\Temp\ExampleEncryptedFile.base64

.. code::

    certutil -decode C:\Temp\ExampleEncryptedFile.base64 C:\Temp\ExampleEncryptedFile


Example: Using an encryption context
####################################

The following examples show how to use an encryption context in a command to encrypt a "hello world" string. 

In AWS KMS, an `encryption context <https://docs.aws.amazon.com/kms/latest/developerguide/encryption-context.html>`_ is a collection of nonsecret name-value pairs that are cryptographically bound to the encrypted data. To decrypt the data, you need to provide the same encryption context. Otherwise, the decrypt operation fails with an ``InvalidCiphertextException`` exception. You can also use the encryption context to `control access to a CMK <https://docs.aws.amazon.com/kms/latest/developerguide/encryption-context.html#encryption-context-authorization>`_ and `identify the operation <https://docs.aws.amazon.com/kms/latest/developerguide/encryption-context.html#encryption-context-auditing>`_ in CloudTrail logs. 

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
    
If you use the JSON format at a Windows command prompt (``cmd.exe``), use a backslash character (\\) to escape all quotation marks inside the curly braces, as shown in the following example.
.. code::

    aws kms encrypt --encryption-context '{\"Dept\": \"IT\"}' --key-id 1234abcd-12ab-34cd-56ef-1234567890ab --plaintext 'hello world' --output text --query CiphertextBlob > C:\Temp\ExampleEncryptedMessage.txt


Part 2: Multivalue encryption context
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
    
If you use the JSON format at a Windows **command prompt** (``cmd.exe``), be sure to use a backslash character (\\) to escape all quotation marks inside the curly braces. For example: 
.. code::

    aws kms encrypt --encryption-context "{\"Dept\": \"IT\",\"Purpose\": \"Test\"}" --key-id 1234abcd-12ab-34cd-56ef-1234567890ab --plaintext "hello world" --output text --query CiphertextBlob > C:\Temp\ExampleEncryptedMessage.txt
    
