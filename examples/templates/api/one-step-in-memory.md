[//]: # "Copyright Amazon.com Inc. or its affiliates. All Rights Reserved."
[//]: # "SPDX-License-Identifier: CC-BY-SA-4.0"

# Basic use of one-step APIs example

Implementations of this example MUST follow the rules defined in
[Example Templates](../../../examples.md#example-templates).

## Implementations

- [Python (DEV)]()
- [Java (DEV)]()

## Definitions

### Conventions used in this document

The key words
"MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT",
"SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL"
in this document are to be interpreted as described in
[RFC 2119](https://tools.ietf.org/html/rfc2119).

## Header

```python
# This example shows how to use the one-step encrypt and decrypt APIs.
#
# In this example, we use an AWS KMS customer master key (CMK),
# but you can use other key management options with the AWS Encryption SDK.
# For examples that demonstrate how to use other key management configurations,
# see the ``keyring`` and ``master_key_provider`` directories.
```

## Summary

```python
# Demonstrate an encrypt/decrypt cycle using the one-step encrypt/decrypt APIs.
```

## Inputs

- **source plaintext** :
  Plaintext to encrypt
- **AWS KMS CMK** :
  The ARN of an AWS KMS CMK that protects data keys

## Steps

1. Define encryption context.

   ```python
   # Prepare your encryption context.
   # Remember that your encryption context is NOT SECRET.
   # https://docs.aws.amazon.com/encryption-sdk/latest/developer-guide/concepts.html#encryption-context
   encryption_context = {
       "encryption": "context",
       "is not": "secret",
       "but adds": "useful metadata",
       "that can help you": "be confident that",
       "the data you are handling": "is what you think it is",
   }
   ```

1. Create keyring.

   ```python
   # Create the keyring that determines how your data keys are protected.
   keyring = AwsKmsKeyring(generator_key_id=aws_kms_cmk)
   ```

1. Encrypt plaintext data.

   ```python
   # Encrypt your plaintext data.
   ciphertext, _encrypt_header = aws_encryption_sdk.encrypt(
       source=source_plaintext, encryption_context=encryption_context, keyring=keyring
   )
   ```

1. Compare ciphertext to plaintext.

   ```python
   # Demonstrate that the ciphertext and plaintext are different.
   assert ciphertext != source_plaintext
   ```

1. Decrypt encrypted data.

   ```python
   # Decrypt your encrypted data using the same keyring you used on encrypt.
   #
   # You do not need to specify the encryption context on decrypt
   # because the header of the encrypted message includes the encryption context.
   decrypted, decrypt_header = aws_encryption_sdk.decrypt(source=ciphertext, keyring=keyring)
   ```

1. Compare the decrypted plaintext and original plaintext.

   ```python
   # Demonstrate that the decrypted plaintext is identical to the original plaintext.
   assert decrypted == source_plaintext
   ```

1. Verify the encryption context.

   ```python
   # Verify that the encryption context used in the decrypt operation includes
   # the encryption context that you specified when encrypting.
   # The AWS Encryption SDK can add pairs, so don't require an exact match.
   #
   # In production, always use a meaningful encryption context.
   assert set(encryption_context.items()) <= set(decrypt_header.encryption_context.items())
   ```