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

Stream desktop applications to users
→ Amazon AppStream 2.0

Create a fast copy of an Aurora database for testing
→ Aurora Cloning
```

---

# AWS Elemental MediaConvert

**AWS Elemental MediaConvert = managed file-based video transcoding service.**

Use it to convert and process **video files for on-demand delivery**.

### Signal

> **File-based video transcoding → MediaConvert**

---

## Example

```text
Video file
    ↓
MediaConvert
    ↓
Transcoded video
```

A company may have a video file and need to produce versions in different formats, codecs, or resolutions for on-demand delivery.

→ **AWS Elemental MediaConvert**

---

## Important distinction

```text
File-based video transcoding
→ MediaConvert

Live video encoding
→ MediaLive
```

### Memory

> **MediaConvert = FILE-BASED VIDEO**

> **MediaLive = LIVE VIDEO**

---

## Legacy service: Amazon Elastic Transcoder

**Amazon Elastic Transcoder was the older managed video/audio transcoding service.**

It was **discontinued on November 13, 2025**.

AWS recommends **MediaConvert** for file-based transcoding workflows.

### SAA memory

> **Video transcoding → MediaConvert**

> **Old question mentioning Elastic Transcoder → recognize it as the legacy service**

### Important

Do not choose Elastic Transcoder for a new modern AWS architecture.

```text
Current file-based video transcoding
→ MediaConvert

Legacy video transcoding service
→ Elastic Transcoder
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
### AWS ParallelCluster

**AWS ParallelCluster** is a service for **deploying and managing HPC (High Performance Computing) clusters on AWS**.

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

| Service            | What it does                         | Signal keyword                |
| ------------------ | ------------------------------------ | ----------------------------- |
| **MediaConvert**   | File-based video transcoding         | Video transcoding             |
| **Artifact**       | AWS compliance documents and reports | Compliance / auditor          |
| **AppStream 2.0**  | Streams desktop applications         | Desktop application streaming |
| **Aurora Cloning** | Fast Aurora database copy            | Aurora copy for testing       |

---

# Important SAA Distinctions

## MediaConvert vs MediaLive

```text
MediaConvert
= file-based video transcoding

MediaLive
= live video encoding
```

So:

> **Video file → MediaConvert**

> **Live video stream → MediaLive**

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

> **"File-based video transcoding for on-demand content."**

→ **AWS Elemental MediaConvert**

---

> **"An application needs to transcode uploaded video files into multiple formats."**

→ **AWS Elemental MediaConvert**

---

> **"An old question mentions Elastic Transcoder."**

→ **Recognize it as the legacy service**

---

> **"Auditors need AWS compliance reports."**

→ **AWS Artifact**

---

> **"A company needs access to AWS SOC/PCI/ISO compliance documentation."**

→ **AWS Artifact**

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

# Specialized Services Decision Tree

```text
What is the requirement?
          │
          ├── File-based video transcoding?
          │       ↓
          │   MediaConvert
          │
          ├── AWS compliance reports/documents?
          │       ↓
          │    Artifact
          │
          ├── Stream desktop applications?
          │       ↓
          │   AppStream 2.0
          │
          └── Fast Aurora database copy?
                  ↓
            Aurora Cloning
```

---

# Pocket Card

| Keyword                                        | Answer                                |
| ---------------------------------------------- | ------------------------------------- |
| File-based video transcoding                   | **MediaConvert**                      |
| On-demand video transcoding                    | **MediaConvert**                      |
| Legacy video transcoding                       | **Elastic Transcoder — discontinued** |
| AWS compliance reports                         | **AWS Artifact**                      |
| SOC / PCI / ISO compliance documents           | **AWS Artifact**                      |
| Stream desktop applications                    | **AppStream 2.0**                     |
| Windows application without local installation | **AppStream 2.0**                     |
| Fast Aurora database copy                      | **Aurora Cloning**                    |
| Aurora copy for testing                        | **Aurora Cloning**                    |

---

# Final Memory

```text
MediaConvert
= FILE-BASED VIDEO TRANSCODING

Elastic Transcoder
= LEGACY / DISCONTINUED

Artifact
= AWS COMPLIANCE DOCUMENTS

AppStream 2.0
= STREAM DESKTOP APPLICATIONS

Aurora Cloning
= FAST AURORA COPY
```

# The Golden Rule

```text
Video file transcoding
→ MediaConvert

AWS compliance reports
→ Artifact

Desktop application streaming
→ AppStream 2.0

Fast Aurora copy
→ Aurora Cloning
```

> **Don't memorize the implementation.**
>
> **Memorize the unique signal.**

For example:

```text
Video transcoding       → MediaConvert
Compliance              → Artifact
Desktop application    → AppStream 2.0
Fast Aurora copy        → Aurora Cloning
```

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
