# Section 24: Directory Services

## The idea

First, what *is* Active Directory?

**Active Directory (AD)** is Microsoft's directory service for centrally managing:

- Users
- Computers
- Groups
- Password authentication
- Permissions
- Group policies

A company might have an AD domain such as:

`corp.example.com`

Windows computers can **join the domain**, which allows users to authenticate with their corporate AD accounts.

Related terms:

- **LDAP** = a protocol used to communicate with directory services
- **Domain join** = connecting a computer to an AD domain
- **Domain Controller (DC)** = a server that runs AD and handles authentication

---

## The AWS problem

A company has workloads in AWS, but those workloads may still need **Microsoft Active Directory** for authentication.

Examples:

- Windows EC2 instances
- Amazon FSx for Windows File Server
- Amazon WorkSpaces
- Amazon RDS for SQL Server
- Applications that use LDAP or Kerberos

AWS provides three main Directory Service options:

1. **AWS Managed Microsoft AD**
2. **AD Connector**
3. **Simple AD**

The main question is:

> **Do I need a real AD in AWS, or do I only need AWS to use an existing AD?**

---

## 1. AWS Managed Microsoft AD

AWS runs a **real Microsoft Active Directory** for you in AWS.

```text
AWS VPC
┌─────────────────────────────┐
│ AWS Managed Microsoft AD    │
│                             │
│ Users                       │
│ Groups                      │
│ Computers                   │
│ Organizational Units        │
│ Group Policies              │
└─────────────────────────────┘
```

It supports standard Microsoft AD functionality such as:

- Domain join
- Group Policy
- LDAP
- Kerberos
- Microsoft AD-compatible applications
- MFA
- Trust relationships with existing AD

You can use it as an **independent AD in AWS**, or connect it to an existing on-premises AD using a trust.

```text
On-premises AD
      │
      │ Trust
      │
AWS Managed Microsoft AD
```

### Important

**Managed Microsoft AD does not mean your on-premises users are automatically copied into AWS.**

The AWS directory and on-premises directory can remain separate while a trust relationship allows supported authentication and access across the two environments.

### Best for

- Need a **real Microsoft AD in AWS**
- Windows workloads need domain joining
- Need full AD features
- Need Group Policy
- Need MFA
- Need a trust with an existing AD
- AWS services require Microsoft AD functionality

### Memory

> **Managed Microsoft AD = real Microsoft AD running in AWS**

---

## 2. AD Connector

**AD Connector does not create another Active Directory.**

It is a **directory gateway/proxy** that connects AWS services to your existing on-premises AD.

```text
AWS service
     │
     ▼
AD Connector
     │
     ▼
On-premises AD
```

The users remain in the existing on-premises AD.

AD Connector forwards authentication requests to that AD and does **not cache directory information in AWS**.

### Example

A company already has:

```text
corp.example.com
```

and wants AWS services to authenticate corporate users without creating another directory.

Use:

**AD Connector**

### Best for

- Existing on-premises AD
- Keep identities on-premises
- AWS services need to authenticate against that AD
- Do not need a separate AD in AWS

### Important limitation

Because authentication depends on the existing AD:

```text
AWS
 ↓
AD Connector
 ↓
On-premises AD
```

AWS needs network connectivity to the on-premises domain controllers.

### Memory

> **AD Connector = connect AWS to an existing AD**

---

## 3. Simple AD

**Simple AD** is a basic directory service based on **Samba** and provides a subset of Microsoft AD functionality.

It can provide:

- Users
- Groups
- Basic domain functionality
- Group Policy
- Kerberos
- EC2 domain joining
- LDAP-compatible functionality

However, it does **not** provide the full capabilities of Microsoft AD.

It does not support features such as:

- Trust relationships
- MFA
- Active Directory Administrative Center
- PowerShell support
- Schema extensions

Simple AD is available in Small and Large sizes, supporting up to approximately **500** and **5,000 users** respectively.

### Important current AWS note

**Simple AD is no longer open to new customers.**

For new deployments, AWS recommends considering **AWS Managed Microsoft AD** or **AD Connector**.

For the SAA exam, however, you should still understand Simple AD because it can appear in questions.

### Best for

- Basic directory requirements
- Small/simple workloads
- No trust requirement
- No MFA requirement
- No advanced Microsoft AD features

### Memory

> **Simple AD = basic Samba-based directory**

---

# The three options compared

| | **Managed Microsoft AD** | **AD Connector** | **Simple AD** |
|---|---|---|---|
| What is it? | Real Microsoft AD in AWS | Directory gateway/proxy | Samba-based AD-compatible directory |
| Directory stored in AWS? | Yes | No | Yes |
| Existing on-premises AD required? | No | **Yes** | No |
| Full Microsoft AD features? | **Yes** | Uses existing AD | No |
| Domain join | Yes | Yes | Yes |
| Group Policy | Yes | Uses on-premises AD | Basic |
| MFA | **Yes** | Yes, through RADIUS | **No** |
| Trust relationships | **Yes** | No | **No** |
| Main purpose | Full AD in AWS | Use existing AD from AWS | Basic/simple directory |
| Users stored in AWS? | Yes, in the AWS directory | **No** | Yes |

---

# The easiest way to choose

### Do I need a real Microsoft AD in AWS?

→ **AWS Managed Microsoft AD**

```text
Users
 ↓
AWS Managed Microsoft AD
 ↓
AWS workloads
```

---

### Do I already have AD on-premises and just want AWS to use it?

→ **AD Connector**

```text
Users
 ↓
On-premises AD
 ↑
AD Connector
 ↑
AWS workloads
```

The users stay in the existing AD.

---

### Do I need only a basic/simple directory?

→ **Simple AD**

```text
Users
 ↓
Simple AD
 ↓
Basic AWS workloads
```

Remember that Simple AD is now closed to new customers.

---

# Trust relationships

A **trust relationship** connects two separate Active Directory environments so that they can recognize identities from each other.

For example:

```text
On-premises AD
corp.example.com
      │
      │ Trust
      │
AWS Managed Microsoft AD
aws.example.com
```

The directories remain separate.

The trust allows supported AWS workloads to authenticate users from the trusted domain.

### Important

**Managed Microsoft AD supports trust relationships.**

**Simple AD does not support trust relationships.**

**AD Connector does not create a trust.** It simply forwards authentication requests to the existing AD.

---

# Common exam patterns

### Existing corporate AD + AWS services

> The company already has an on-premises AD and wants AWS services to authenticate against it without storing directory information in AWS.

→ **AD Connector**

---

### Full AD in AWS

> Windows EC2 instances must join a domain and the company needs full Microsoft AD features in AWS.

→ **AWS Managed Microsoft AD**

---

### Trust with on-premises AD

> AWS workloads need to use identities from an existing corporate AD through a trust relationship.

→ **AWS Managed Microsoft AD**

---

### MFA

> The company requires MFA for its directory.

→ **AWS Managed Microsoft AD** or **AD Connector with RADIUS**

Simple AD does not support MFA.

---

### Basic/low-cost directory

> A small workload needs a basic LDAP-compatible directory and does not need trust or MFA.

→ **Simple AD**

For new customers, remember that Simple AD is no longer available for new deployments.

---

# AWS Console access with corporate AD

This is a common exam pattern.

Suppose:

- Developers already exist in the company's **on-premises AD**
- They are already organized into AD groups
- The company wants them to access the **AWS Management Console**
- Access must be **role-based**

The important pieces are:

```text
Corporate AD
     ↓
AD Connector
     ↓
Federation
     ↓
IAM Role
     ↓
AWS Console
```

Here:

**AD Connector**

→ allows AWS to use the existing corporate identities.

**IAM Role**

→ defines what those users are allowed to do in AWS.

Do not confuse:

- **AD groups** = groups in the corporate directory
- **IAM roles** = AWS permissions
- **IAM groups** = groups for IAM users, not the normal solution for federated corporate identities

## Corporate users accessing AWS

Corporate users such as Alice and Bob are stored in the company's **Active Directory**, not as IAM users.

Federation allows those corporate identities to access AWS without creating a separate IAM user for each person.

```text
Corporate AD
├── Alice
├── Bob
└── Developers
        ↓
   Federation
        ↓
       AWS
        ↓
IAM Role / IAM Identity Center
        ↓
AWS account & resources

```
The roles are on the AWS side, not inside Active Directory.

AD → Who is the user?
Federation → Connects the corporate identity to AWS
IAM Role / IAM Identity Center → What can the user do in AWS?
AWS account/resources → What they ultimately access
Important

AD users are not AWS accounts.

Alice → corporate AD user
AWS account → separate AWS environment
IAM role → AWS permissions
### Exam clue

> Existing corporate AD + federation + role-based AWS Console access

→ **AD Connector + IAM Roles**

---

# Key traps

### Trap 1

> "Create a trust with on-premises AD"

→ **Managed Microsoft AD**

Simple AD does not support trust relationships.

### Trap 2

> "Keep identities only on-premises"

→ **AD Connector**

It does not store directory information in AWS.

### Trap 3

> "Need a full Microsoft AD in AWS"

→ **Managed Microsoft AD**

### Trap 4

> "Basic LDAP + small/simple workload"

→ **Simple AD**

### Trap 5

> "Existing AD + AWS Console + role-based access"

→ **AD Connector + IAM Roles**

---

# Pocket card

| Requirement | Answer |
|---|---|
| Full Microsoft AD in AWS | **Managed Microsoft AD** |
| Existing on-premises AD | **AD Connector** |
| Keep identities on-premises | **AD Connector** |
| Trust relationship | **Managed Microsoft AD** |
| Full AD features | **Managed Microsoft AD** |
| MFA | **Managed Microsoft AD** or AD Connector with RADIUS |
| Basic/legacy directory | **Simple AD** |
| Simple AD trust | **Not supported** |
| AWS Console via corporate AD | **AD Connector + IAM Roles** |

## Memory hook

**Managed Microsoft AD**  
→ **Real AD in AWS**

**AD Connector**  
→ **Connect to existing AD**

**Simple AD**  
→ **Basic Samba directory**
