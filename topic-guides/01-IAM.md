# Section 1: IAM — Identity and Access Management

## The idea

IAM controls **who can access AWS resources and what they are allowed to do**.

IAM is:

* **Global** — not tied to a Region.
* **Free** — there is no additional charge for IAM itself.
* Based on **identities, policies, and credentials**.

The most important distinction:

> **Authentication = Who are you?**
> **Authorization = What are you allowed to do?**

IAM mainly handles authorization, while authentication can come from IAM itself or from an external identity system through **federation**.

---

## Users, groups, and roles

| Identity      | What it is                                                     | Typical use                                                      |
| ------------- | -------------------------------------------------------------- | ---------------------------------------------------------------- |
| **IAM User**  | A specific AWS identity with long-term credentials             | Individual AWS identities when federation is not being used      |
| **IAM Group** | A collection of IAM users                                      | Give multiple users the same permissions                         |
| **IAM Role**  | An identity that is assumed and provides temporary credentials | AWS services, applications, federation, and cross-account access |

### IAM user

An IAM user can have:

* Console password
* Access keys
* Permissions through identity-based policies

Access keys are **long-lived credentials**, so they should not be used when a role can be used instead.

### IAM group

A group is only a collection of **IAM users**.

A group cannot contain:

* another group
* a role

Groups are mainly used to manage permissions for multiple IAM users.

### IAM role

A role does not represent one permanently logged-in person.

A principal **assumes the role**, and AWS provides temporary credentials.

Common examples:

```text
EC2 → IAM role
Lambda → execution role
ECS task → task role
User → assumed role
Account A → role in Account B
Federated user → IAM role
```

### Golden rule

> **AWS workload needs AWS permissions → use an IAM role, not hard-coded access keys.**

For example:

```text
EC2
 ↓
IAM instance role
 ↓
Temporary credentials
 ↓
S3 / DynamoDB / SQS / etc.
```

Do not:

```text
EC2
 ↓
Hard-coded access key
 ↓
S3
```

The same principle applies to Lambda, ECS, and other AWS services that support IAM roles.

---

## IAM policies

Policies are JSON documents that define permissions.

Basic structure:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject"
      ],
      "Resource": "arn:aws:s3:::company-bucket/reports/*"
    }
  ]
}
```

### Important policy elements

| Element       | Meaning                                                           |
| ------------- | ----------------------------------------------------------------- |
| **Effect**    | `Allow` or `Deny`                                                 |
| **Action**    | API operations such as `s3:GetObject`                             |
| **Resource**  | The AWS resource affected, usually identified by an ARN           |
| **Condition** | Optional conditions such as MFA, source IP, VPC, TLS, tags, etc.  |
| **Principal** | Who the policy applies to; mainly seen in resource-based policies |

### Example

```json
{
  "Effect": "Allow",
  "Action": "s3:GetObject",
  "Resource": "arn:aws:s3:::company-bucket/alice/*"
}
```

This grants access to objects under:

```text
s3://company-bucket/alice/
```

but not:

```text
s3://company-bucket/bob/
```

---

## Policy types

| Policy type               | Attached to                             | Purpose                                                    |
| ------------------------- | --------------------------------------- | ---------------------------------------------------------- |
| **Identity-based policy** | User, group, or role                    | Defines what that identity can do                          |
| **Resource-based policy** | Resource such as S3 bucket or SQS queue | Defines who can access the resource                        |
| **Permissions boundary**  | IAM user or role                        | Maximum permissions that identity can have                 |
| **SCP**                   | AWS Organizations account or OU         | Maximum permissions allowed in the account                 |
| **Session policy**        | Role session                            | Further restricts permissions for a specific session       |
| **S3 ACL**                | S3 bucket/object                        | Legacy access-control mechanism; generally prefer policies |

### Grant vs ceiling

This distinction is heavily tested:

* **Identity policy** → can grant permissions.
* **Resource policy** → can grant permissions.
* **Permissions boundary** → limits maximum permissions; grants nothing by itself.
* **SCP** → limits maximum permissions; grants nothing by itself.
* **Session policy** → further limits a role session.

Think:

```text
Identity permissions
        ∩
Permissions boundary
        ∩
SCP
        ∩
Session policy
```

An applicable explicit Deny can block the request.

---

## How AWS evaluates permissions

The basic rules are:

1. **Everything starts as implicitly denied.**
2. An applicable **Allow** can grant access.
3. An applicable **explicit Deny always wins**.

Common reasons an apparently allowed request fails:

* Explicit Deny
* SCP
* Permissions boundary
* Session policy
* Resource policy restrictions
* Incorrect trust policy when assuming a role

### Important distinction

A policy saying:

```text
Allow s3:GetObject
```

does not necessarily mean the request will succeed.

Another applicable policy can still deny it.

---

# STS and temporary credentials

**AWS Security Token Service (STS)** provides temporary AWS credentials.

Temporary credentials consist of:

* Access key ID
* Secret access key
* Session token

They expire automatically.

The most important STS operation for the exam is:

```text
sts:AssumeRole
```

Typical flow:

```text
Principal
   ↓
AssumeRole
   ↓
AWS STS
   ↓
Temporary credentials
   ↓
Access AWS resources
```

### Why use temporary credentials?

They:

* expire automatically
* reduce the need for long-lived access keys
* are suitable for applications, federation, and cross-account access

A role session can have a configured duration within AWS-supported limits; the exact maximum depends on the type of role/session and configuration. For exam questions, focus primarily on **temporary vs long-lived credentials** rather than memorizing one universal duration.

---

# IAM roles have two important policies

A role has two separate concepts that must not be confused.

## 1. Permissions policy

Defines:

> **What can the role do?**

Example:

```text
Allow:
s3:GetObject
s3:PutObject
```

## 2. Trust policy

Defines:

> **Who can assume the role?**

Example:

```text
EC2 service
Account A
SAML identity provider
OIDC identity provider
```

This is a critical exam distinction.

### Example

A role might allow:

```text
s3:GetObject
```

but its trust policy might only allow:

```text
ecs-tasks.amazonaws.com
```

An EC2 instance cannot simply use that role.

### Exam pattern

> "The role has the required permissions, but the principal cannot assume it."

Check the:

**Trust policy**

---

# Federation

Federation allows users to authenticate using an **external identity system** instead of creating a separate IAM user for every person.

Typical external identity systems include:

* Active Directory
* Microsoft Entra ID
* Okta
* Other corporate identity providers

The important idea is:

> **The corporate directory authenticates the user; AWS provides authorization through roles and temporary credentials.**

---

## Direct federation with AWS

This is especially important for questions involving:

* corporate AD/LDAP
* SSO
* temporary AWS credentials
* no IAM user for every employee

Typical architecture:

```text
Corporate AD / LDAP
        ↓
Identity Provider (IdP)
        ↓
SAML / federation
        ↓
AWS STS
        ↓
IAM Role
        ↓
Temporary credentials
        ↓
AWS resources
```

The external identity provider confirms the user's identity.

AWS then gives the user temporary credentials associated with an IAM role.

### Important

The corporate directory does **not** become an AWS IAM user database containing 1,200 IAM users.

Instead:

```text
1,200 corporate users
        ↓
External identity system
        ↓
Federation
        ↓
IAM roles
        ↓
Temporary AWS credentials
```

This is one of the most important federation patterns for SAA.

---

# Federation + S3 per-user folders

A common exam scenario is:

> A company has hundreds or thousands of employees in corporate AD/LDAP. Each employee should access their own folder in an S3 bucket. The company does not want to create an IAM user for every employee.

The solution uses:

```text
Corporate AD / LDAP
        ↓
Identity Provider / Federation
        ↓
STS
        ↓
IAM Role
        ↓
IAM Policy
        ↓
S3 prefix
```

For example:

```text
s3://company-documents/alice/
s3://company-documents/bob/
s3://company-documents/john/
```

The authorization policy can restrict Alice to:

```text
company-documents/alice/*
```

while Bob can access:

```text
company-documents/bob/*
```

### Exam pattern

> "1200 employees already exist in corporate AD/LDAP. They need S3 access and SSO. Each user should access only their own folder."

Think:

**Federation + STS + IAM role/policy + S3 prefix**

Do **not** automatically create 1,200 IAM users.

---

# Federation protocols

Two important federation technologies appear in AWS questions.

## SAML 2.0

Common for:

```text
Corporate workforce
        ↓
AD / Entra ID / Okta
        ↓
SAML
        ↓
AWS
```

SAML is commonly associated with **enterprise workforce SSO**.

## OIDC

OpenID Connect is commonly used for:

* web/mobile authentication
* workloads
* GitHub Actions and other external systems
* Kubernetes/EKS workload identity scenarios

For SAA, remember the broad distinction:

```text
Enterprise workforce SSO → commonly SAML
Modern application/workload federation → commonly OIDC
```

---

# IAM Identity Center

**IAM Identity Center** is AWS's workforce SSO service.

It is particularly useful when employees need access to:

* multiple AWS accounts
* AWS applications
* business applications

Typical architecture:

```text
Corporate IdP
      ↓
IAM Identity Center
      ↓
AWS accounts
      ↓
Permission sets
      ↓
IAM roles
```

It can integrate with external identity providers such as:

* Microsoft Entra ID
* Okta
* other SAML-compatible IdPs
* Active Directory environments

### Signal

> "Employees need one login to access multiple AWS accounts."

Think:

**IAM Identity Center**

---

# Direct federation vs IAM Identity Center

Do not treat these as the same thing.

### Direct federation

The application/workflow directly uses federation and STS to obtain temporary AWS credentials.

```text
Corporate IdP
    ↓
Federation
    ↓
STS
    ↓
IAM role
    ↓
AWS resources
```

This is especially relevant to scenarios such as:

> "Corporate AD users need temporary AWS credentials to access an S3 bucket."

### IAM Identity Center

AWS provides a centralized workforce SSO experience:

```text
Corporate IdP
    ↓
IAM Identity Center
    ↓
AWS accounts
    ↓
Permission sets / roles
```

This is especially relevant to:

> "Employees need SSO access to multiple AWS accounts."

### Exam signal

| Question wording                                        | Likely answer            |
| ------------------------------------------------------- | ------------------------ |
| Corporate users need temporary AWS credentials          | Federation + STS         |
| Existing AD users should access AWS without IAM users   | Federation               |
| One login to many AWS accounts                          | IAM Identity Center      |
| Centralized employee access across AWS accounts         | IAM Identity Center      |
| S3 access for thousands of corporate users              | Federation + roles + STS |
| External application users sign in with Google/Facebook | Cognito                  |

---

# IAM Identity Center vs Cognito

These are commonly confused.

## IAM Identity Center

For:

**Employees / workforce users**

```text
Company employee
      ↓
Corporate IdP
      ↓
AWS accounts
```

## Amazon Cognito

For:

**Application users / customers**

Example:

```text
Mobile app
    ↓
Cognito
    ↓
Application user
```

If a question says:

> Customers sign in to a web/mobile application using Google, Facebook, Apple, or username/password.

Think:

**Cognito**

If it says:

> Employees need SSO into AWS accounts.

Think:

**IAM Identity Center**

---

# Cross-account access

Suppose:

```text
Account A = auditors
Account B = production
```

Auditors in Account A need temporary access to Account B.

The standard solution is:

### Account B

Create a role with the required permissions.

### Account B trust policy

Trust Account A.

### Account A

Allow its users to call:

```text
sts:AssumeRole
```

### Result

```text
Account A user
      ↓
AssumeRole
      ↓
STS
      ↓
Temporary credentials
      ↓
Role in Account B
      ↓
Account B resources
```

### Exam signal

> "Users from another AWS account need temporary access."

Think:

**Cross-account IAM role + trust policy + STS**

Do not create shared IAM users or exchange long-lived access keys.

---

# ExternalId and third-party access

When a third-party AWS account assumes your role, the trust policy can require an:

**ExternalId**

This helps protect against the **confused deputy problem**.

Typical scenario:

```text
Your AWS account
      ↓
Role
      ↑
Third-party SaaS / partner
```

The third party provides the expected ExternalId when assuming the role.

### Exam signal

> "Third-party service needs to assume a role on behalf of customers."

Think:

**ExternalId**

---

# Permission boundaries

A permissions boundary defines the **maximum permissions** an IAM user or role can receive.

It does not grant permissions by itself.

Example:

```text
Identity policy
      +
Permissions boundary
      =
Effective permissions
```

If the identity policy allows:

```text
s3:*
```

but the boundary only allows:

```text
s3:GetObject
```

the identity cannot use the permissions outside the boundary.

### Exam signal

> "Developers can create roles, but those roles must never exceed a predefined maximum."

Think:

**Permissions boundary**

---

# Service Control Policies (SCPs)

SCPs are part of:

**AWS Organizations**

They define the maximum available permissions for accounts or organizational units.

An SCP:

* does not grant permissions
* restricts what an account can do
* can affect IAM users and roles
* can restrict the account root user

Example:

```text
Organization
    ↓
OU
    ↓
Account
    ↓
SCP denies certain services
```

Even if an IAM policy says:

```text
Allow
```

the SCP can still prevent the action.

### Exam signal

> "The account is in AWS Organizations and an action is blocked despite an IAM Allow."

Think:

**SCP**

---

# Permissions boundary vs SCP

This distinction is important.

|                             | Permissions boundary                | SCP                                       |
| --------------------------- | ----------------------------------- | ----------------------------------------- |
| Scope                       | One IAM user/role                   | AWS account / OU                          |
| Grants permissions?         | No                                  | No                                        |
| Purpose                     | Maximum permissions for an identity | Maximum permissions allowed in an account |
| AWS Organizations required? | No                                  | Yes                                       |

Think:

```text
Permissions boundary → one identity
SCP                  → whole account / OU
```

---

# MFA

MFA adds another authentication factor.

A policy can require MFA using:

```text
aws:MultiFactorAuthPresent
```

Example concept:

```text
Allow EC2 termination
ONLY IF
MFA is present
```

### Exam signal

> "Users must provide MFA before performing a sensitive action."

Think:

**`aws:MultiFactorAuthPresent`**

---

# Root user

The root user is the identity associated with the AWS account's original email address.

Root has extremely broad permissions and should not be used for normal administration.

Best practices:

* Enable MFA.
* Do not create root access keys.
* Do not use root for daily work.
* Use an appropriate administrative identity instead.
* Use root only for tasks that specifically require it.

### Important SCP nuance

An SCP can restrict the root user in member accounts of an AWS Organization.

The management account is different; SCPs do not restrict the permissions of its root user in the same way.

---

# IAM Access Analyzer and credential tools

## IAM Access Analyzer

Helps identify resources that are accessible from outside the intended trust boundary.

Typical exam wording:

> "Which IAM tool identifies resources shared with external principals?"

Answer:

**IAM Access Analyzer**

## IAM credentials report

Provides information about IAM users and their credentials, such as:

* password usage
* access key age
* MFA status

Useful for finding stale credentials.

## IAM access advisor

Shows service permissions/usage information that can help identify unused permissions.

---

# S3-specific IAM concepts

S3 commonly uses both identity-based and resource-based policies.

### Identity policy

Attached to:

```text
User / Group / Role
```

Example:

```text
Role
 ↓
Allow s3:GetObject
```

### Bucket policy

Attached to:

```text
S3 bucket
```

It can specify:

```text
Principal
```

For example:

```text
Principal = specific IAM role
```

Bucket policies are especially important for:

* cross-account access
* restricting access to specific principals
* enforcing S3 security requirements

---

# S3 prefixes and per-user access

S3 does not have traditional folders in the same way a filesystem does.

What looks like a folder is generally an object key prefix.

Example:

```text
company/
  alice/
    document1.pdf
    document2.pdf

  bob/
    document3.pdf
```

Policies can restrict users to a particular prefix.

For example:

```text
arn:aws:s3:::company/alice/*
```

This pattern commonly appears together with federation.

### Full architecture

```text
Corporate AD / LDAP
        ↓
Identity Provider
        ↓
Federation
        ↓
STS
        ↓
IAM Role
        ↓
IAM Policy
        ↓
S3 prefix
```

This is the exact pattern to recognize when a question combines:

* corporate directory
* SSO
* temporary credentials
* S3
* per-user folders

---

# S3 ACLs

S3 ACLs are an older access-control mechanism.

For modern S3 architectures:

> Prefer IAM policies and S3 bucket policies.

Do not confuse:

```text
S3 ACL
```

with:

```text
VPC Network ACL
```

They are completely different.

* **S3 ACL** → S3 access control
* **Network ACL** → subnet-level network filtering

---

# Common exam patterns

> **"An EC2 instance needs access to S3."**
> → Attach an **IAM role** to the EC2 instance.

> **"Lambda needs permission to read DynamoDB."**
> → Use a **Lambda execution role**.

> **"Credentials are hard-coded in an application on EC2."**
> → Replace them with an **IAM role**.

> **"A role has the correct permissions but cannot be assumed."**
> → Check the **trust policy**.

> **"Users in Account A need temporary access to Account B."**
> → **Cross-account role + trust policy + STS AssumeRole**.

> **"A third-party SaaS provider needs to assume your role."**
> → Consider **ExternalId** in the trust policy.

> **"An IAM policy allows the action, but the action is denied."**
> → Look for an **explicit Deny, SCP, permissions boundary, session policy, or resource-policy restriction**.

> **"Developers can create roles, but those roles must never exceed a defined permission set."**
> → **Permissions boundary**.

> **"An account in AWS Organizations cannot perform an action even though IAM allows it."**
> → Check the **SCP**.

> **"5,000 employees already have corporate AD credentials and need AWS access without creating thousands of IAM users."**
> → **Federation / IAM Identity Center**, depending on the architecture described.

> **"Corporate users need temporary AWS credentials to access S3."**
> → **Federation + STS + IAM role**.

> **"Employees need one login to access many AWS accounts."**
> → **IAM Identity Center**.

> **"Customers sign in to an application using Google or Facebook."**
> → **Amazon Cognito**.

> **"Each corporate employee should only access their own S3 folder."**
> → Use **IAM policies restricting access to the user's S3 prefix**, often combined with federation.

> **"A sensitive action should only work after MFA."**
> → Condition key **`aws:MultiFactorAuthPresent`**.

> **"What should be done first to secure a new AWS account?"**
> → **Protect the root user with MFA, avoid root access keys, and use a proper administrative identity for normal work.**

---

# Pocket card

| Keyword                                    | Think                                 |
| ------------------------------------------ | ------------------------------------- |
| EC2 needs AWS access                       | IAM role                              |
| Lambda needs AWS access                    | Execution role                        |
| ECS task needs AWS access                  | Task role                             |
| Temporary credentials                      | STS                                   |
| `sts:AssumeRole`                           | Assume a role                         |
| Role has permissions but cannot be assumed | Trust policy                          |
| What can the role do?                      | Permissions policy                    |
| Who can assume the role?                   | Trust policy                          |
| Cross-account access                       | Role + trust policy + STS             |
| Third-party role assumption                | ExternalId                            |
| Maximum permissions for one identity       | Permissions boundary                  |
| Maximum permissions for an AWS account/OU  | SCP                                   |
| External corporate users                   | Federation                            |
| Corporate AD/LDAP → AWS                    | Federation / IdP                      |
| Federation → temporary AWS credentials     | STS                                   |
| One login → multiple AWS accounts          | IAM Identity Center                   |
| Enterprise SSO with SAML                   | IAM Identity Center / SAML federation |
| Application customer login                 | Cognito                               |
| Per-user S3 folder                         | S3 prefix + IAM policy                |
| Policy attached to S3 bucket               | Bucket policy                         |
| Legacy S3 permissions                      | S3 ACL                                |
| Explicit Deny                              | Always wins                           |
| MFA requirement                            | `aws:MultiFactorAuthPresent`          |
| New AWS account                            | Secure root + MFA                     |
| External resource sharing analysis         | IAM Access Analyzer                   |
| Stale IAM credentials                      | Credentials report                    |
| Unused permissions                         | Access advisor                        |

---

# Core mental model

Most IAM questions can be reduced to four questions:

### 1. Who is requesting access?

```text
IAM user
Role
AWS service
Federated user
Cross-account principal
```

### 2. How did they authenticate?

```text
IAM credentials
Federation
SAML
OIDC
AssumeRole
```

### 3. What grants or restricts the access?

```text
Identity policy
Resource policy
Permissions boundary
SCP
Session policy
```

### 4. Is there an explicit Deny?

```text
Yes → Denied
No  → Continue evaluating
```

The most important federation pattern to memorize is:

```text
Corporate AD / LDAP
        ↓
Identity Provider
        ↓
SAML / Federation
        ↓
AWS STS
        ↓
IAM Role
        ↓
Temporary credentials
        ↓
AWS resource
```

And when the resource is S3:

```text
IAM Role
   ↓
IAM Policy
   ↓
S3 bucket/prefix
   ↓
User's designated objects
```

**Do not create an IAM user for every employee just because the employees already exist in a corporate directory. Federation exists specifically to avoid that pattern.**
