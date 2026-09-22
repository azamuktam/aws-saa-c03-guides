# Section 24: Directory Services

## The idea

**Active Directory (AD)** is Microsoft's directory service for centrally managing:

* Users
* Computers
* Groups
* Password authentication
* Permissions
* Group Policies

Example domain:

```text
corp.example.com
```

Related terms:

* **LDAP** → protocol used by directory services
* **Domain join** → connect a computer to an AD domain
* **Domain Controller (DC)** → server running AD and handling authentication

AWS provides three main options:

1. **AWS Managed Microsoft AD**
2. **AD Connector**
3. **Simple AD**

The key question:

> **Need a real AD in AWS, or only access to an existing AD?**

---

# AWS Managed Microsoft AD

Runs a **real Microsoft Active Directory in AWS**.

Supports:

* Domain join
* Group Policy
* LDAP
* Kerberos
* Microsoft AD-compatible applications
* MFA
* Trust relationships with existing AD

```text
AWS VPC
└── Managed Microsoft AD
    ├── Users
    ├── Groups
    ├── Computers
    ├── OUs
    └── Group Policies
```

Can be:

* Independent AD in AWS
* Connected to on-premises AD through a **trust**

```text
On-prem AD
    │
   Trust
    │
Managed Microsoft AD
```

### Important

Managed Microsoft AD does **not automatically copy on-premises users into AWS**. The directories can remain separate while a trust allows supported authentication/access.

### Best for

* Real Microsoft AD in AWS
* Domain joining Windows workloads
* Full AD features
* Group Policy
* MFA
* Trust with existing AD
* AWS services requiring Microsoft AD functionality

> **Managed Microsoft AD = real Microsoft AD in AWS**

---

# AD Connector

**AD Connector = bridge/proxy to an existing AD.**

It does **not create another AD** and does **not cache directory information in AWS**.

```text
AWS service
    ↓
AD Connector
    ↓
On-prem AD
```

Users remain in the existing on-premises AD.

### Best for

* Existing on-premises AD
* Keep identities on-premises
* AWS services authenticate against existing AD
* No separate AD required in AWS

Requires network connectivity from AWS to the on-premises domain controllers.

> **AD Connector = connect AWS to an existing AD**

---

# Simple AD

**Simple AD = basic Samba-based, AD-compatible directory.**

Provides:

* Users
* Groups
* Basic domain functionality
* Group Policy
* Kerberos
* EC2 domain joining
* LDAP-compatible functionality

Does **not** provide full Microsoft AD capabilities such as:

* Trust relationships
* MFA
* Active Directory Administrative Center
* PowerShell support
* Schema extensions

Sizes:

* **Small** → up to ~500 users
* **Large** → up to ~5,000 users

### Current AWS note

Simple AD is **closed to new customers**. For new deployments, consider **Managed Microsoft AD** or **AD Connector**.

> **Simple AD = basic Samba directory**

---

# The three options compared

|                               | **Managed Microsoft AD** | **AD Connector**             | **Simple AD**         |
| ----------------------------- | ------------------------ | ---------------------------- | --------------------- |
| What is it?                   | Real Microsoft AD        | Gateway/proxy to existing AD | Samba-based directory |
| Directory stored in AWS?      | **Yes**                  | **No**                       | **Yes**               |
| Existing on-prem AD required? | No                       | **Yes**                      | No                    |
| Full Microsoft AD features?   | **Yes**                  | Uses existing AD             | No                    |
| Domain join                   | Yes                      | Yes                          | Yes                   |
| Group Policy                  | Yes                      | Uses on-prem AD              | Basic                 |
| MFA                           | **Yes**                  | Yes, with RADIUS             | **No**                |
| Trust relationships           | **Yes**                  | No                           | **No**                |
| Main purpose                  | Full AD in AWS           | Use existing AD from AWS     | Basic directory       |
| Users stored in AWS?          | Yes                      | **No**                       | Yes                   |

---

# Choosing the service

### Need a real AD in AWS?

→ **AWS Managed Microsoft AD**

```text
Users
 ↓
Managed Microsoft AD
 ↓
AWS workloads
```

### Already have on-premises AD and want AWS to use it?

→ **AD Connector**

```text
Users
 ↓
On-prem AD
 ↑
AD Connector
 ↑
AWS workloads
```

Users stay in the existing AD.

### Need only a basic directory?

→ **Simple AD**

Remember: no new Simple AD deployments for new customers.

---

# Trust relationships

A **trust** connects separate AD environments so supported identities can be recognized across them.

```text
On-prem AD
corp.example.com
      │
     Trust
      │
AWS Managed AD
aws.example.com
```

Directories remain separate.

### Important

* **Managed Microsoft AD** → supports trusts
* **Simple AD** → no trusts
* **AD Connector** → does not create a trust; forwards authentication to existing AD

---

# Common exam patterns

> **Existing corporate AD + AWS services + identities remain on-premises**
> → **AD Connector**

> **Full Microsoft AD in AWS**
> → **Managed Microsoft AD**

> **AWS workloads need identities from an existing AD through a trust**
> → **Managed Microsoft AD**

> **MFA required**
> → **Managed Microsoft AD** or **AD Connector + RADIUS**

> **Basic/low-cost LDAP-compatible directory**
> → **Simple AD**

> **Simple AD trust required**
> → **Not supported**

---

# AWS Console + corporate AD

For existing corporate users accessing AWS with role-based permissions:

```text
Corporate AD
     ↓
AD Connector
     ↓
Federation
     ↓
IAM Role / IAM Identity Center
     ↓
AWS
```

### Important distinction

* **AD groups** → corporate directory groups
* **IAM roles** → AWS permissions
* **IAM groups** → groups for IAM users, not the normal solution for federated corporate identities

Corporate users do not need to become IAM users.

```text
Alice
→ Corporate AD user

AWS account
→ Separate AWS environment

IAM Role
→ AWS permissions
```

### Mental model

```text
AD
→ Who is the user?

Federation
→ Connect corporate identity to AWS

IAM Role / IAM Identity Center
→ What can the user do?

AWS resources
→ What they access
```

### Exam clue

> Existing corporate AD + AWS Console + role-based access

→ **AD Connector + IAM Roles**

---

# Key traps

> **"Create a trust with on-prem AD."**
> → **Managed Microsoft AD**

> **"Keep identities only on-premises."**
> → **AD Connector**

> **"Need a full Microsoft AD in AWS."**
> → **Managed Microsoft AD**

> **"Basic LDAP + small/simple workload."**
> → **Simple AD**

> **"Existing AD + AWS Console + role-based access."**
> → **AD Connector + IAM Roles**

---

# Pocket card

| Requirement                  | Answer                                                |
| ---------------------------- | ----------------------------------------------------- |
| Full Microsoft AD in AWS     | **Managed Microsoft AD**                              |
| Existing on-prem AD          | **AD Connector**                                      |
| Keep identities on-premises  | **AD Connector**                                      |
| Trust relationship           | **Managed Microsoft AD**                              |
| Full AD features             | **Managed Microsoft AD**                              |
| MFA                          | **Managed Microsoft AD** or **AD Connector + RADIUS** |
| Basic/legacy directory       | **Simple AD**                                         |
| Simple AD trust              | **Not supported**                                     |
| AWS Console via corporate AD | **AD Connector + IAM Roles**                          |


