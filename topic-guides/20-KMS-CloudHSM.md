# Section 20: KMS & CloudHSM

## The idea

**AWS Key Management Service (KMS)** manages cryptographic keys used to protect data.

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

# KMS key deletion protection

KMS deletion is deliberately hard to do by accident. There is **no single "delete" button** — deletion always goes through a **scheduled, cancellable process**.

## DeleteKey vs ScheduleKeyDeletion

| Operation | What it does |
|---|---|
| **`ScheduleKeyDeletion`** | The normal API call to delete a KMS key. Puts the key into **PendingDeletion** with a waiting period of **7–30 days**. |
| **`DeleteKey`** | The **underlying CloudTrail event name** you monitor for. In practice, when someone "deletes" a key, it appears as a scheduled deletion action. |
| **`CancelKeyDeletion`** | Cancels a pending deletion **during the waiting period** and returns the key to a **Disabled** state. This is the "undo" action. |

```text
ScheduleKeyDeletion
        ↓
Key enters PendingDeletion (7–30 days)
        ↓
   ┌────┴────┐
   ↓         ↓
CancelKey    Waiting period ends
Deletion         ↓
   ↓         Key permanently deleted
Key becomes
Disabled
(reversible)
```

## KMS key states

| State | Meaning | Reversible? |
|---|---|---|
| **Enabled** | Key works normally | — |
| **Disabled** | Key cannot perform crypto operations | ✅ Yes (re-enable) |
| **PendingDeletion** | Scheduled for deletion, waiting period active | ✅ Yes (CancelKeyDeletion) |
| **PendingImport** | Key has no material; awaiting import | ✅ Yes (import) |
| **Unavailable** | Key temporarily not usable (e.g., custom key store issue) | Depends on cause |

Key point:

> **Disabling and PendingDeletion both stop the key from working — but both are reversible.**

## Preventing accidental deletion (exam pattern)

The classic scenario:

> "Protect KMS keys from accidental deletion **and** alert admins via email, with minimal operational overhead."

Best answer:

```text
Amazon EventBridge rule  → detects DeleteKey / ScheduleKeyDeletion events
        ↓
   ┌────┴─────────────────────────┐
   ↓                              ↓
SNS topic notifies          Systems Manager Automation
administrators by email     runbook runs CancelKeyDeletion
                            (undoes the pending deletion)
```

Why this is best:

* **EventBridge** detects the deletion attempt.
* **Systems Manager Automation** (`AWSConfigRemediation-CancelKeyDeletion` runbook) automatically cancels the deletion — no custom Lambda code.
* **SNS** emails administrators.
* This is an **AWS Prescriptive Guidance pattern** with a ready-made CloudFormation template → **minimal operational overhead**.

### Distractors to recognize

| Distractor | Why it's wrong |
|---|---|
| **CloudTrail → CloudWatch Logs → metric filter → SNS** | Only **alerts**; does **not prevent** deletion. |
| **AWS Config rule to "reverse" deletion** | AWS Config evaluates compliance; it does not reverse KMS API actions. |
| **Custom Lambda to block deletion** | Works, but **more operational overhead** than the built-in SSM runbook. |

### Exam signal

> "Prevent accidental KMS key deletion AND alert admins with least overhead."

→ **EventBridge + Systems Manager Automation (CancelKeyDeletion) + SNS**

> "Immediately stop a key from being used."

→ **Disable** (not delete)

> "Permanently remove a key."

→ **ScheduleKeyDeletion (7–30 day waiting period)**

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

CloudHSM gives you more direct control than KMS:

* Dedicated HSMs
* Customer-controlled HSM users
* Customer-controlled key material
* HSM cluster management
* Cryptographic operations inside the HSM
* HSM access from applications in your VPC

```text
KMS
→ managed key service
→ AWS manages the HSM infrastructure

CloudHSM
→ dedicated HSM
→ customer manages the HSM cluster
→ customer controls keys inside the HSM
```

AWS cannot view or perform cryptographic operations with your CloudHSM keys.

---

# CloudHSM backups

CloudHSM automatically creates **periodic cluster backups at least every 24 hours**.

Backups contain encrypted copies of:

* Users
* Key material
* Certificates
* HSM configuration
* Policies

The backup is encrypted by the HSM before the data leaves the HSM. AWS cannot decrypt the backup because AWS does not have access to the key needed to decrypt it.

```text
CloudHSM
   ↓
Encrypted cluster backup
   ↓
AWS stores backup
```

The default backup retention period is **90 days**; the supported retention range is **7–379 days**.

### Restoring from a backup

A CloudHSM backup can be used to create a **new cluster** containing the users, key material, certificates, configuration, and policies from that backup.

```text
CloudHSM backup
      ↓
Create new cluster from backup
      ↓
Keys + users + configuration restored
```

You can also copy CloudHSM backups to another Region for disaster recovery.

### Important distinction

The backup represents a specific recovery point.

Therefore:

> Data created or modified **after the latest recoverable backup** can be lost.

---

# CloudHSM zeroization

**Zeroization destroys the key material, certificates, and other data currently stored on the HSM.**

A zeroized HSM does **not automatically mean every key is permanently lost**, because a valid CloudHSM backup may still exist.

```text
Zeroization
    ↓
Current HSM data destroyed
    ↓
Recoverable backup exists?
    ↓
Yes → Restore from backup
No  → Key material is unrecoverable
```

AWS explicitly states that data created or modified after the most recent backup is lost and unrecoverable if an HSM is zeroized.

### SAA exam rule

> **CloudHSM key + no recoverable backup/copy → permanently lost**

> **CloudHSM key + valid backup → restore from backup**

> **AWS Support cannot provide your plaintext CloudHSM keys**

### Practice-question trap

A common practice question says:

> "The HSM was zeroized and you did not have a copy of the keys."

The intended answer is:

→ **The keys are lost permanently.**

The important concept is **no recoverable copy**.

Current CloudHSM also automatically maintains cluster backups, so the real-world question should be interpreted as meaning that **no usable backup containing those keys exists**.

---

# CloudHSM login / zeroization distinction

Do not confuse **account lockout** with **HSM zeroization**.

Current CloudHSM CLI behavior:

* More than **5 incorrect login attempts** locks the account.
* An administrator can reset the user's password.
* With multiple HSMs, additional failed attempts may occur because the client load-balances requests across HSMs.

Zeroization is a separate destructive operation.

AWS also documents that an HSM can be zeroized without authentication if someone can access the HSM through the relevant network path.

### Exam distinction

```text
Wrong password
→ account lockout

Zeroization
→ destroys current HSM data
```

---

# CloudHSM vs KMS

|                                   | **KMS**                                   | **CloudHSM**                 |
| --------------------------------- | ----------------------------------------- | ---------------------------- |
| Main purpose                      | Managed key management                    | Dedicated HSM                |
| Infrastructure                    | AWS-managed                               | Customer-controlled cluster  |
| HSM tenancy                       | AWS-managed shared service infrastructure | **Single-tenant HSMs**       |
| Key control                       | KMS controls key operations               | Customer controls HSM keys   |
| AWS service integration           | **Excellent / built-in**                  | More application-specific    |
| Key material exposed to customer? | No                                        | Keys remain inside HSM       |
| Customer manages HSM users        | No                                        | **Yes**                      |
| Backup/recovery                   | Managed by KMS                            | **CloudHSM cluster backups** |
| Custom HSM-level control          | Limited                                   | **High**                     |

### Decision rule

> "Need simple AWS-service encryption and key management."

→ **KMS**

> "Need dedicated HSMs and direct control over HSM/key operations."

→ **CloudHSM**

> "Need a standard managed AWS encryption service."

→ **KMS**

> "Need hardware-based cryptographic processing with customer control."

→ **CloudHSM**

---

# Question patterns

> **Encrypt large files with KMS** → **Envelope encryption / GenerateDataKey**

> **User has `kms:*` but receives AccessDenied** → **Check KMS key policy**

> **Need customer-controlled key policy / rotation / disable / deletion** → **Customer managed KMS key**

> **Immediately stop a KMS key from being used** → **Disable key**

> **Permanently delete a KMS key** → **Schedule deletion, 7–30 days**

> **Prevent accidental KMS key deletion + alert admins (least overhead)** → **EventBridge + Systems Manager Automation (CancelKeyDeletion) + SNS**

> **Undo a pending KMS key deletion** → **CancelKeyDeletion (key returns to Disabled)**

> **Monitor for KMS key deletion attempts** → **EventBridge rule on `DeleteKey` / `ScheduleKeyDeletion`**

> **Multi-Region application needs related KMS keys** → **Multi-Region KMS key**

> **SSE-KMS causes excessive KMS requests/costs** → **S3 Bucket Key**

> **Need dedicated HSMs in your VPC** → **CloudHSM**

> **CloudHSM needs disaster recovery** → **CloudHSM backups / cross-Region backup copy**

> **CloudHSM HSM is zeroized + valid backup exists** → **Restore from backup / create cluster from backup**

> **CloudHSM key is lost + no recoverable backup** → **Key is permanently lost**

> **CloudHSM + AWS Support asked for plaintext keys** → **AWS cannot provide them**

> **More than 5 incorrect CloudHSM login attempts** → **Account is locked**

---

# Pocket card

| Keyword                                     | Answer                            |
| ------------------------------------------- | --------------------------------- |
| Managed AWS key service                     | **KMS**                           |
| Dedicated HSM in your VPC                   | **CloudHSM**                      |
| AWS-controlled service encryption key       | **AWS managed key**               |
| Customer-controlled KMS key                 | **Customer managed key**          |
| No customer key control                     | **AWS owned key**                 |
| KMS authorization issue                     | **Check key policy**              |
| Large data + KMS                            | **Envelope encryption**           |
| Generate data-encryption key                | **`GenerateDataKey`**             |
| Stop KMS key immediately                    | **Disable**                       |
| Permanently remove KMS key                  | **Schedule deletion, 7–30 days**  |
| Delete a KMS key (API call)                 | **`ScheduleKeyDeletion`**         |
| CloudTrail event for KMS deletion           | **`DeleteKey`**                   |
| Undo a pending KMS deletion                 | **`CancelKeyDeletion`**           |
| Key state after cancelling deletion         | **Disabled**                      |
| Key state during deletion waiting period    | **PendingDeletion**               |
| Prevent accidental KMS deletion + alert     | **EventBridge + SSM + SNS**       |
| Customer-managed symmetric key rotation     | **Automatic rotation**            |
| Multi-Region KMS                            | **Multi-Region key**              |
| Reduce SSE-KMS requests/cost                | **S3 Bucket Key**                 |
| Dedicated HSM + customer control            | **CloudHSM**                      |
| CloudHSM periodic backup                    | **At least every 24 hours**       |
| CloudHSM default backup retention           | **90 days**                       |
| CloudHSM backup retention range             | **7–379 days**                    |
| Restore CloudHSM                            | **Create cluster from backup**    |
| Zeroized HSM + backup exists                | **Restore from backup**           |
| Zeroized HSM + no recoverable backup        | **Key permanently lost**          |
| CloudHSM backup plaintext accessible to AWS | **No**                            |
| >5 incorrect CloudHSM logins                | **Account locked**                |
| CloudHSM DR across Regions                  | **Copy backup to another Region** |
