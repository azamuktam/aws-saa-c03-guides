# Section 20: KMS & CloudHSM

## The idea

**AWS Key Management Service (KMS)** manages cryptographic keys used to protect data.

The main questions in KMS are:

* Where is the key material stored?
* Who can use the key?
* Who controls the key policy?
* Can the key be rotated?
* Can the key be disabled or deleted?
* Can key usage be audited?
* Do you need AWS-managed key infrastructure or dedicated HSM infrastructure?

**CloudHSM** is different from ordinary KMS because you get **single-tenant HSMs** in your VPC and direct control over the HSM cluster and its key material.

The most important distinction:

```text
KMS
→ managed key-management service
→ easy AWS-service integration
→ AWS operates the underlying HSM infrastructure

CloudHSM
→ dedicated HSMs
→ customer manages the HSM cluster
→ customer controls the keys inside the HSMs
```

KMS itself stores key material inside AWS KMS HSMs and does not expose AWS-generated key material through the KMS API. Current AWS KMS HSMs use FIPS 140-3 Security Level 3-compliant hardware in standard AWS Regions.

---

# KMS key types

There are three important categories:

| Key type                 | Who manages it? | Customer control | Typical use                                                                |
| ------------------------ | --------------- | ---------------- | -------------------------------------------------------------------------- |
| **AWS owned key**        | AWS service     | None             | Encryption by AWS services with no customer key-management overhead        |
| **AWS managed key**      | AWS KMS         | Limited          | Convenient encryption for a specific AWS service                           |
| **Customer managed key** | Customer        | High             | When you need control over policies, permissions, rotation, disable/delete |

## AWS owned keys

AWS owns and manages these keys.

You generally do not see or manage them.

Important characteristics:

* No monthly KMS key fee
* No customer key policy
* Customer cannot rotate them
* Customer cannot delete them
* Customer cannot audit their key activity directly

Use them when:

> "Encryption is required, but we do not need customer control over the key."

---

## AWS managed keys

AWS creates these keys in your account for use by an AWS service.

Examples:

```text
aws/s3
aws/rds
aws/ebs
```

You can see the key in KMS, but AWS controls its lifecycle.

Important points:

* No monthly fee for the key
* Automatically rotated by AWS approximately every year
* You cannot change the rotation schedule
* You cannot change the key policy in the same way you can with a customer managed key
* Useful when you need KMS-backed encryption without managing the key yourself

---

## Customer managed keys

You create and manage these keys yourself.

You control:

* Key policy
* IAM permissions
* Grants
* Enable/disable state
* Deletion scheduling
* Rotation configuration
* Key aliases
* Key usage permissions

Customer managed keys are the answer when the question emphasizes:

> **"We need control over the encryption key."**

---

# Key policy and IAM permissions

Every KMS key has a **key policy**.

The key policy is a resource-based policy attached directly to the KMS key.

Unlike ordinary IAM permissions, KMS has an important additional rule:

> An IAM policy cannot grant access to a KMS key unless the key policy allows the account to use IAM policies for that key.

This is why simply giving someone:

```text
kms:*
```

in IAM does not automatically guarantee access.

The effective authorization depends on:

* Key policy
* IAM policy
* Grants
* Explicit Denies
* Other applicable policy controls

### Important exam pattern

> "The user has `kms:*`, but KMS returns AccessDenied."

→ **Check the KMS key policy first.**

The default KMS key policy normally includes a statement that allows IAM policies in the account to control access. But a custom key policy can remove that path.

---

# KMS cryptographic operations

KMS supports several types of cryptographic keys, including:

* Symmetric encryption keys
* Asymmetric encryption keys
* HMAC keys

For the common symmetric KMS key used by AWS services:

> KMS directly encrypts only small amounts of data, up to **4 KB**.

For large data, do not send the entire file to KMS.

Use **envelope encryption**.

---

# Envelope encryption

Envelope encryption means:

> Use a KMS key to protect a smaller data key, and use the data key to encrypt the actual large data.

Example:

```text
1. Application → KMS
   GenerateDataKey

2. KMS returns:
   - Plaintext data key
   - Encrypted data key

3. Application:
   - Uses plaintext data key to encrypt the large file
   - Stores encrypted data key with the ciphertext
   - Discards plaintext data key

4. Decryption:
   - Send encrypted data key to KMS
   - KMS decrypts the data key
   - Use plaintext data key to decrypt the file
```

Conceptually:

```text
KMS key
   ↓
encrypts
   ↓
Data key
   ↓
encrypts
   ↓
Large data
```

The large file does not have to pass through KMS.

AWS explicitly uses this pattern because KMS is intended to protect data keys rather than directly encrypt large amounts of application data.

### Exam signal

> "Encrypt a 100 MB file with KMS."

→ **Envelope encryption / GenerateDataKey**

Not:

→ `kms:Encrypt` on the entire file.

---

# Key rotation

Key rotation creates new cryptographic key material while keeping the same KMS key identity.

For supported customer managed symmetric KMS keys:

```text
Same KMS key ID
Same KMS key ARN
New cryptographic key material
```

This means applications usually do not need to change which KMS key they reference.

Automatic rotation:

* Is optional for customer managed keys.
* Defaults to **365 days** when enabled.
* Can now use a configurable period from **90 to 2560 days** for supported keys.
* Is automatically performed by AWS for AWS managed keys every year.

Automatic rotation is **not supported** for asymmetric keys, HMAC keys, imported key material, or KMS keys in custom key stores; those can be rotated manually where supported.

### Exam signal

> "Customer must control the rotation of a symmetric KMS key."

→ **Customer managed key**

> "AWS automatically rotates the service-specific KMS key."

→ **AWS managed key**

---

# Disabling vs deleting a KMS key

These are very different.

## Disable

Disabling a KMS key:

* Happens immediately.
* Is reversible.
* Prevents cryptographic operations using the key.

Use this when:

> "Stop use of the key immediately."

## Delete

KMS key deletion is not immediate.

You **schedule deletion**, and AWS requires a waiting period of **7–30 days**.

During this period:

```text
Pending deletion
       ↓
Can cancel deletion
```

After the waiting period:

```text
KMS key permanently deleted
```

### Exam signal

> "Immediately stop all use of a compromised KMS key."

→ **Disable the key**

Not delete it.

### Important exception: imported key material

If a customer imported key material into KMS, the **key material itself can be manually deleted** without deleting the KMS key metadata.

This is different from AWS-generated key material.

---

# Multi-Region KMS keys

A multi-Region key is a related set of KMS keys in different AWS Regions that share:

* Same key ID
* Same key material

This allows related keys to be used interchangeably for cryptographic operations across Regions.

Example:

```text
us-east-1
KMS primary
     │
     └── same key material
             │
             ▼
eu-west-1
KMS replica
```

Data encrypted with one related key can be decrypted using another related key without a cross-Region KMS call.

### Typical use cases

* Disaster recovery
* Multi-Region applications
* Globally distributed data
* Client-side encryption spanning Regions

### Important limitations

Multi-Region keys are **not one global KMS key**.

Each Region has its own KMS key resource.

Also:

> **You cannot create a multi-Region key in a custom key store.**

---

# S3 Bucket Keys

S3 can use **S3 Bucket Keys** with SSE-KMS.

Without a bucket key:

```text
S3 object
   ↓
KMS request
```

For many objects, this can generate a large number of KMS requests.

With an S3 Bucket Key:

```text
S3 bucket
   ↓
Bucket-level key
   ↓
Many objects
```

This reduces the number of KMS requests made by S3 and can reduce KMS request costs.

The trade-off is that CloudTrail logging becomes less granular because KMS is not necessarily called for every individual object encryption operation.

### Exam signal

> "SSE-KMS is generating too many KMS requests / KMS costs are high."

→ **Enable S3 Bucket Key**

---

# AWS CloudHSM

**AWS CloudHSM** provides dedicated, single-tenant HSMs that run in your VPC.

HSM means:

**Hardware Security Module**

An HSM is specialized hardware designed to securely generate, store, and use cryptographic keys.
