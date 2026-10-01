# Section 37H: Specialized Services & Governance

## The idea

These are smaller AWS services that usually appear in SAA questions as **specific use cases** that do not fit neatly into the main service families.

You generally don't need deep knowledge of each one.

The best strategy is:

> **Read the requirement → identify the unique keyword → choose the service.**

For example:

```text
File-based video transcoding
→ AWS Elemental MediaConvert

AWS compliance reports
→ AWS Artifact

Software license limits
→ AWS License Manager

Share AWS resources across accounts
→ AWS Resource Access Manager (RAM)

Stream desktop applications to users
→ Amazon AppStream 2.0

Create a fast copy of an Aurora database for testing
→ Aurora Cloning
```

---

# AWS Artifact

**AWS Artifact = access AWS compliance documents and reports.**

It is a document portal for AWS compliance information.

Examples include AWS compliance documentation such as:

* SOC
* PCI
* ISO-related reports

### Signal

> **Auditor needs AWS compliance reports → Artifact**

---

## Example

> "An auditor needs access to AWS compliance reports for the company's audit."

→ **AWS Artifact**

---

## Important distinction

Artifact is a **document portal**.

It is not:

* a monitoring service
* a logging service
* a security monitoring service
* a compliance monitoring engine

Think:

```text
AWS compliance documents
        ↓
    AWS Artifact
```

### Memory

> **Artifact = COMPLIANCE DOCUMENTS**

---

# AWS License Manager

**AWS License Manager = track and control software licenses used by AWS resources.**

Use it when a company has a limited number of software licenses, such as **Windows Server licenses for EC2**.

### Signal

> **Software licenses + track usage + enforce license limit → AWS License Manager**

---

## Example

A company has **50 Windows Server licenses**.

The licenses are counted based on the **vCPU count** of EC2 instances.

```text
50 licenses
    ↓
AWS License Manager
    ↓
Track EC2 license usage
    ↓
Limit reached?
 ├─ No → EC2 launch allowed
 └─ Yes → Prevent additional launch
```

Enable **license limit enforcement** to prevent usage from exceeding the available licenses.

**Amazon SNS** can be used for notifications.

### Memory

> **Software licenses + EC2 + enforce license limit → License Manager**

---

# AWS Resource Access Manager (RAM)

**AWS RAM = share supported AWS resources across AWS accounts.**

It is commonly used in **multi-account AWS environments**.

### Signal

> **Share AWS resources across accounts → AWS RAM**

---

## Example

A company has multiple AWS accounts and wants them to use a shared **VPC subnet** or **Transit Gateway**.

```text
AWS Account A
      │
      ├── Share resource
      ↓
    AWS RAM
      ↓
AWS Account B
```

### Examples

RAM can be used to share supported resources such as:

* VPC subnets
* Transit Gateways
* Route 53 Resolver rules

### Memory

> **RAM = SHARE AWS RESOURCES ACROSS ACCOUNTS**

---

### AWS ParallelCluster

**AWS ParallelCluster = deploy and manage HPC (High Performance Computing) clusters on AWS.**

Commonly used with:

* **EC2** → compute nodes
* **Slurm** → job scheduler
* **FSx for Lustre** → high-performance shared storage
* **EFS** → shared file storage

Typical use cases:

* Scientific simulations
* Engineering workloads
* Genomics
* Weather modeling
* Large-scale numerical processing

**Remember:**

> **ParallelCluster = deploy and manage HPC clusters**

---

# Amazon AppStream 2.0

**Amazon AppStream 2.0 = stream desktop applications to users from AWS.**

The application runs on AWS, while the user accesses it remotely through a browser or compatible client.

### Typical pattern

```text
User's laptop
     ↓
   Browser
     ↓
AppStream 2.0
     ↓
Application runs on AWS
```

---

## Use it when

You need to give users access to applications without requiring the applications to be installed locally on their computers.

Common situations include:

* desktop applications
* Windows applications
* centralized application access
* remote application streaming

### Signal

> **Users need to access a desktop application without installing it locally → AppStream 2.0**

---

## Example

> "Users need to access a Windows application from their laptops without installing the application locally."

→ **Amazon AppStream 2.0**

---

## Important distinction

AppStream 2.0 streams **applications** to users.

Think:

```text
Application runs in AWS
        ↓
Stream the application
        ↓
User interacts remotely
```

### Memory

> **AppStream 2.0 = STREAM DESKTOP APPLICATIONS**

---

# Aurora Cloning

**Aurora Cloning = create a fast copy of an Amazon Aurora database for testing, development, or other purposes.**

It allows you to create a new Aurora cluster from an existing Aurora cluster without initially making a full physical copy of all the underlying data.

This makes cloning much faster and more storage-efficient than a traditional full database copy.

### Signal

> **Create a fast copy of an Aurora database for testing → Aurora Cloning**

---

## Example

A production Aurora database contains a large amount of data.

Developers need a separate environment for testing.

Instead of creating a complete physical copy:

```text
Production Aurora
       ↓
   Aurora Clone
       ↓
Testing / Development
```

→ **Aurora Cloning**

---

## Why use it?

Typical use cases include:

* testing
* development
* experimentation
* creating temporary database environments

### Memory

> **Aurora Cloning = FAST AURORA COPY**

---

# Specialized Services Comparison

| Service             | What it does                                   | Signal keyword                       |
| ------------------- | ---------------------------------------------- | ------------------------------------ |
| **Artifact**        | AWS compliance documents and reports           | Compliance / auditor                 |
| **License Manager** | Tracks and enforces software licenses          | Software license limit               |
| **AWS RAM**         | Shares supported AWS resources across accounts | Resource sharing / multiple accounts |
| **ParallelCluster** | Deploys and manages HPC clusters               | HPC / High Performance Computing     |
| **AppStream 2.0**   | Streams desktop applications                   | Desktop application streaming        |
| **Aurora Cloning**  | Fast Aurora database copy                      | Aurora copy for testing              |

---

## Artifact vs CloudWatch

Do not confuse compliance documentation with monitoring.

```text
Artifact
= AWS compliance documents

CloudWatch
= metrics + logs + alarms
```

If the requirement is for an auditor to obtain AWS compliance reports:

→ **Artifact**

---

## License Manager vs AWS RAM

```text
License Manager
= Manage / enforce software licenses

AWS RAM
= Share supported AWS resources
```

---

## AppStream 2.0 vs normal application deployment

AppStream is not simply a service for deploying a web application.

The key requirement is:

> **Users remotely access a desktop application that runs in AWS.**

```text
Desktop application
→ AppStream 2.0
```

---

## Aurora Cloning vs snapshot restore

These can both create another database environment, but the question signal differs.

```text
Fast copy of an existing Aurora cluster
→ Aurora Cloning
```

A traditional snapshot-based workflow is different and involves restoring from a snapshot.

For SAA, remember the unique signal:

> **Fast Aurora copy for testing/development → Aurora Cloning**

---

# Common Question Patterns

> **"Auditors need AWS compliance reports."**

→ **AWS Artifact**

---

> **"A company needs access to AWS SOC/PCI/ISO compliance documentation."**

→ **AWS Artifact**

---

> **"A company has 50 Windows Server licenses and wants to stop EC2 launches when all licenses are used."**

→ **AWS License Manager**

---

> **"Software licenses are counted based on EC2 vCPUs and the company wants to enforce the license limit."**

→ **AWS License Manager**

---

> **"A company has multiple AWS accounts and wants to share a VPC subnet between them."**

→ **AWS Resource Access Manager (RAM)**

---

> **"A company wants to share a Transit Gateway with other AWS accounts."**

→ **AWS Resource Access Manager (RAM)**

---

> **"Users need to access a Windows application without installing it locally."**

→ **Amazon AppStream 2.0**

---

> **"Employees need to use a desktop application remotely from their browsers."**

→ **Amazon AppStream 2.0**

---

> **"Create a fast copy of an Aurora database for testing."**

→ **Aurora Cloning**

---

> **"Developers need an isolated copy of a large Aurora database for development."**

→ **Aurora Cloning**

---

# Pocket Card

| Keyword                                        | Answer                                |
| ---------------------------------------------- | ------------------------------------- |
| File-based video transcoding                   | **MediaConvert**                      |
| On-demand video transcoding                    | **MediaConvert**                      |
| Legacy video transcoding                       | **Elastic Transcoder — discontinued** |
| AWS compliance reports                         | **AWS Artifact**                      |
| SOC / PCI / ISO compliance documents           | **AWS Artifact**                      |
| Software license limits                        | **AWS License Manager**               |
| Windows Server licenses on EC2                 | **AWS License Manager**               |
| Enforce software license limit                 | **AWS License Manager**               |
| Share resources across AWS accounts            | **AWS RAM**                           |
| Share VPC subnets across accounts              | **AWS RAM**                           |
| Share Transit Gateway across accounts          | **AWS RAM**                           |
| HPC cluster                                    | **ParallelCluster**                   |
| Stream desktop applications                    | **AppStream 2.0**                     |
| Windows application without local installation | **AppStream 2.0**                     |
| Fast Aurora database copy                      | **Aurora Cloning**                    |
| Aurora copy for testing                        | **Aurora Cloning**                    |

---

# Section 37 Complete

The full **Section 37: Gap-Fill Services** is now divided into:

```text
37A → Migration, Messaging & Disaster Recovery

37B → Certificates, DNS, IPv6 & Edge Infrastructure

37C → Data Lakes, ETL & Data Integration

37D → Analytics, Big Data & Streaming

37E → AI & Machine Learning One-Liners

37F → Systems Manager & Operations

37G → Application Development & Integration

37H → Specialized Services & Governance
```
