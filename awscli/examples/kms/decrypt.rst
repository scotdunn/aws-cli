The following command shows the recommended way to decrypt data with the AWS CLI.

.. code::

    aws kms decrypt --ciphertext-blob fileb://ExampleEncryptedFile --output text --query Plaintext | base64 --decode > ExamplePlaintextFile

The command does several things:

#. Uses the ``fileb://`` prefix in the value of the ``--ciphertext-blob`` parameter.

    The ``fileb://`` prefix tells the CLI to get the plaintext data from a binary file. If the file is not in the current directory, type the full path to file. For example: ``fileb:///var/tmp/ExampleEncryptedFile`` or ``fileb://C:\Temp\ExampleEncryptedFile``.

    For more information about reading AWS CLI parameter values from a file, see `Loading Parameters from a File <https://docs.aws.amazon.com/cli/latest/userguide/cli-using-param.html#cli-using-param-file>`_ in the *AWS Command Line Interface User Guide* and `Best Practices for Local File Parameters <https://blogs.aws.amazon.com/cli/post/TxLWWN1O25V1HE/Best-Practices-for-Local-File-Parameters>`_ on the AWS Command Line Tool Blog.

    The command assumes the ciphertext in ``ExampleEncryptedFile`` is binary data. The `encrypt examples <encrypt.html#examples>`_ demonstrate how to save a ciphertext this way.

#. Uses the ``--output`` and ``--query`` parameters to control the command's output.

    These parameters select the CiphertextBlob field of the response, which contains the encrypted data, and returns the output as text, rather than JSON. For more information about controlling output, see `Controlling Command Output <https://docs.aws.amazon.com/cli/latest/userguide/controlling-output.html>`_ in the *AWS Command Line Interface User Guide*.

#. Uses the ``base64`` utility.

    This utility decodes the extracted plaintext to binary data. The plaintext that is returned by a successful ``decrypt`` command is base64-encoded text. You must decode this text to obtain the original plaintext.

#. Saves the binary plaintext to a file.

    The final part of the command (``> ExamplePlaintextFile``) saves the binary plaintext data to a file.

    
**Example: Using an encryption context**

The following command uses an encryption context in a command to decrypt the contents of the ExampleEncryptedMessage file. In AWS KMS, an `encryption context <https://docs.aws.amazon.com/kms/latest/developerguide/encryption-context.html>`_ is a collection of non-secret name-value pairs that are cryptographically bound to the encrypted data. 

In AWS KMS, when an encryption context was provided in the command to encrypt data, you must provide the same encryption context in the command to decrypt the data. If the command to decrypt this data does not include the same encryption context, the Decrypt operation fails with an ``InvalidCiphertextException`` exception.

In this example, the encryption context consists of two name-value pairs, ``Dept=IT`` and ``Purpose=Test``, separated by a comma. This example uses the shorthand syntax, but you can use JSON syntax or specify the path to a file that contains the encryption context in JSON or shorthand syntax. You do not need to use the same syntax in the Encrypt and Decrypt commands.

.. code::

    aws kms decrypt --ciphertext-blob fileb://ExampleEncryptedMessage --encryption-context Dept=IT,Purpose=Test --output text --query Plaintext | base64 --decode > ExamplePlaintextMessage

If you use the JSON format at a Windows command prompt, be sure to use a backslash character (\) to escape all quotation marks inside the curly braces. For example: 

.. code::

    aws kms decrypt --ciphertext-blob fileb://ExampleEncryptedMessage --encryption-context '{\"Dept\": \"IT\",\"Purpose\": \"Test\"}' --output text --query Plaintext | base64 --decode > ExamplePlaintextMessage
    
**Example: Using the AWS CLI to decrypt data from the Windows command prompt**

The preceding example assumes the ``base64`` utility is available, which is commonly the case on Linux and Mac OS X. For the Windows command prompt, use ``certutil`` instead of ``base64``. This requires two commands, as shown in the following examples.

You can also use the `Invoke-KMSDecrypt <https://docs.aws.amazon.com/powershell/latest/reference/items/Invoke-KMSDecrypt.html>`_ cmdlet in the AWSPowerShell module on Windows, macOS, or Linux systems.

.. code::

    aws kms decrypt --ciphertext-blob fileb://ExampleEncryptedFile --output text --query Plaintext > ExamplePlaintextFile.base64

.. code::

    certutil -decode ExamplePlaintextFile.base64 ExamplePlaintextFile