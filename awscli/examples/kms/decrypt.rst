Example: Decrypting data
########################

The following command shows the recommended way to decrypt data with the AWS CLI. This command decrypts the binary data in the ExampleEncryptedFile, selects the plaintext data from the response, Base64-decodes it, and saves it in a file.

This command does not specify an AWS KMS customer master key (CMK) because KMS can identify the CMK from the encrypted data.

.. code::

    aws kms decrypt --ciphertext-blob fileb://ExampleEncryptedFile --output text --query Plaintext | base64 --decode > ExamplePlaintextFile

About this command:

* Gets the ciphertext from a file.

    The value of the ``plaintext`` parameter is a file path with the ``fileb://`` prefix. The ``b`` in the prefix tells the CLI that the files contains binary data.

    For more information about reading AWS CLI parameter values from a file, see `Loading Parameters from a File <https://docs.aws.amazon.com/cli/latest/userguide/cli-using-param.html#cli-using-param-file>`_ in the *AWS Command Line Interface User Guide* and `Best Practices for Local File Parameters <https://blogs.aws.amazon.com/cli/post/TxLWWN1O25V1HE/Best-Practices-for-Local-File-Parameters>`_ on the AWS Command Line Tool Blog.

* Selects the plaintext data from the response.

    The ``query`` parameter selects the ``Plaintext`` field of the response, which contains the decrypted data. The ``output`` parameter returns the output as text, rather than JSON. For more information about controlling output, see `Controlling Command Output <https://docs.aws.amazon.com/cli/latest/userguide/controlling-output.html>`_ in the *AWS Command Line Interface User Guide*.

* Decodes the plaintext data.

    All fields in an AWS CLI response contain Base64-encoded text. Before you can decrypt the data, you must decode this text. This command uses the ``base64`` utility to convert the Base64-encoded text to binary data.

* Saves the plaintext to a file.

    The final part of the command (``> ExamplePlaintextFile``) saves the plaintext data to a file.


Example: Encrypting data in Windows (cmd.exe)
#############################################

To Base64-encode or decode data in a ``cmd.exe`` window, use ``certutil``. In this example, the first command decrypts the data and writes the plaintext to a file. The second command Base64-decodes the data and writes it to a different file.

You can also use the `Invoke-KMSDecrypt <https://docs.aws.amazon.com/powershell/latest/reference/items/Invoke-KMSDecrypt.html>`_ cmdlet in the AWSPowerShell module on Windows, macOS, or Linux systems.

.. code::

    aws kms decrypt --ciphertext-blob fileb://ExampleEncryptedFile --output text --query Plaintext > ExamplePlaintextFile.base64

.. code::

    certutil -decode ExamplePlaintextFile.base64 ExamplePlaintextFile



Example: Using an encryption context
####################################

This example shows how to use an encryption context in a command to decrypt data. In AWS KMS, an `encryption context <https://docs.aws.amazon.com/kms/latest/developerguide/encryption-context.html>`_ is a collection of non-secret name-value pairs that are cryptographically bound to the encrypted data. When an encryption context is included in the command to encrypt data, you must provide the same encryption context in the command to decrypt the data. Otherwise, the Decrypt operation fails with an ``InvalidCiphertextException`` exception.

In this example, the encryption context consists of two name-value pairs, ``Dept=IT`` and ``Purpose=Test``. This example uses the shorthand syntax, but you can use JSON syntax or specify the path to a file that contains the encryption context in JSON or shorthand syntax. For a more detailed example that shows all formats, see the `Example: Using an encryption context <https://github.com/juneb/aws-cli/blob/kms-examples/awscli/examples/kms/encrypt.rst#example-using-an-encryption-context>`_. You do not need to use the same syntax in the ``encrypt`` and ``decrypt`` commands.

.. code::

    aws kms decrypt --encryption-context Dept=IT,Purpose=Test --ciphertext-blob fileb://ExampleEncryptedMessage --output text --query Plaintext | base64 --decode > ExamplePlaintextMessage

If you use the JSON format in a Windows command prompt (``cmd.exe``), use a backslash character (\\) to escape all quotation marks inside the curly braces, as shown in the following example.

.. code::

    aws kms decrypt --encryption-context '{\"Dept\": \"IT\",\"Purpose\": \"Test\"}' --ciphertext-blob fileb://ExampleEncryptedMessage --output text --query Plaintext | base64 --decode > ExamplePlaintextMessage
    
