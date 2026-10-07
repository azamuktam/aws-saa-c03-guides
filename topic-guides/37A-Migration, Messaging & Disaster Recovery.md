# Section 37A: Migration, Messaging & Disaster Recovery

## The idea

These AWS services commonly appear in SAA questions for **migration, messaging, database migration, backups, and disaster recovery**.

Best approach:

> **Read the requirement → identify the unique keyword → choose the service.**

```text
Existing RabbitMQ / ActiveMQ application
→ Amazon MQ

Lift-and-shift / rehost servers to AWS
→ AWS Transform MGN
  (formerly AWS Application Migration Service)

Backup and restore AWS resources
→ AWS Backup

Disaster recovery for servers
→ AWS Elastic Disaster Recovery (DRS)

Migrate database data
→ AWS Database Migration Service (DMS)

Different database engines
→ AWS Schema Conversion Tool (SCT) + DMS

Customer engagement / targeted messaging
→ Amazon Pinpoint
  (legacy SAA knowledge; support ends Oct 30, 2026)

Direct SMS / voice / push messaging
→ AWS End User Messaging
```

---

# Amazon MQ

**Amazon MQ = managed message broker for applications already using traditional messaging systems.**

Managed brokers include:

* ActiveMQ
* RabbitMQ

Use it when an existing application already depends on traditional messaging protocols/APIs and you want to migrate to AWS with minimal application changes.

### Common keywords

* RabbitMQ
* ActiveMQ
* JMS
* AMQP
* Existing message broker
* Minimal application changes
* Existing broker-based application

### Important distinction

```text
Existing traditional broker
→ Amazon MQ

New AWS-native application
→ SQS / SNS / EventBridge
```

### Example

> "An existing application uses RabbitMQ and the company wants to migrate to AWS with minimal code changes."

→ **Amazon MQ**

### Memory

> **Existing message broker → Amazon MQ**

---

# Amazon Pinpoint

**Amazon Pinpoint = customer engagement and targeted multichannel messaging service.**

Historically, Pinpoint was used for:

* Email campaigns
* SMS campaigns
* Push notifications
* Customer journeys
* Audience segmentation
* Targeted marketing
* Engagement/event tracking

### Important current-status note

AWS has announced that **Amazon Pinpoint will end support on October 30, 2026**. After that date, the Pinpoint console and Pinpoint resources such as campaigns, journeys, segments, and analytics will no longer be available.

However, AWS has moved messaging capabilities such as **SMS, voice, and mobile push** into **AWS End User Messaging**.

### For SAA exam questions

Older practice questions may still use:

```text
Targeted customer engagement
+ campaigns / journeys / segmentation
→ Amazon Pinpoint
```

So **know Pinpoint for legacy SAA questions**, but also recognize the current AWS service direction.

### Current AWS messaging model

```text
SMS / MMS / voice
→ AWS End User Messaging SMS

Push notifications
→ AWS End User Messaging Push

OTP / verification
→ AWS End User Messaging Notify
```

### Important distinction

```text
Customer engagement / historical campaign platform
→ Amazon Pinpoint

Direct messaging APIs
→ AWS End User Messaging
```

### Memory

> **Legacy targeted customer engagement → Amazon Pinpoint**
> **Current direct messaging → AWS End User Messaging**

---

# AWS Transform MGN

**AWS Transform MGN = rehost / lift-and-shift servers to AWS.**

> **Former name: AWS Application Migration Service**

AWS renamed Application Migration Service to **AWS Transform MGN** in June 2026. The underlying MGN replication capabilities remain the same.

MGN continuously replicates the source server's **block-level data** to AWS and can migrate physical, virtual, and cloud servers with minimal downtime.

```text
On-premises server
        ↓
MGN replication
        ↓
Continuous block-level replication
        ↓
AWS staging area
        ↓
Test / cutover
        ↓
EC2
```

## Use it when

> "Move existing physical or virtual servers to AWS without redesigning the application."

→ **AWS Transform MGN**

### Common keywords

* Rehost
* Lift-and-shift
* Existing physical servers
* Existing virtual servers
* Move applications to AWS
* Minimal application changes
* Minimal downtime
* Continuous block-level replication

### Example

> "A company needs to rehost physical servers to AWS while minimizing business interruption."

→ **AWS Transform MGN**

### Memory

> **MGN = move servers**

---

# AWS Elastic Disaster Recovery (DRS)

**AWS Elastic Disaster Recovery (DRS) = disaster recovery for servers.**

It continuously replicates workloads into AWS so they can be recovered during a disaster or business continuity event.

```text
Production server
      ↓
Continuous replication
      ↓
AWS recovery environment

Disaster
   ↓
Launch recovery instances
```

## Use it when

> "We need disaster recovery for physical, virtual, or cloud servers."

→ **AWS Elastic Disaster Recovery (DRS)**

DRS focuses on **recovery**, not simply completing a migration project. It supports recovery instances, drill instances, recovery plans, and failback.

### Memory

```text
DRS
= Disaster Recovery
= recover servers in AWS
```

---

# MGN vs DRS

Both services use continuous replication, so SAA questions can make them look very similar.

The key difference is the **objective**:

```text
Need to MOVE the workload
→ AWS Transform MGN

Need to RECOVER the workload after a disaster
→ AWS Elastic Disaster Recovery (DRS)
```

### Comparison

| Service                                 | Main purpose                    | Typical signal                               |
| --------------------------------------- | ------------------------------- | -------------------------------------------- |
| **AWS Transform MGN**                   | Rehost / migrate servers to AWS | Lift-and-shift / migration / minimal changes |
| **AWS Elastic Disaster Recovery (DRS)** | Disaster recovery for servers   | Disaster / business continuity / recovery    |

### Mental model

```text
Migration project
→ MGN

Business continuity / disaster recovery
→ DRS
```

### Important exam trap

Do **not** choose based only on:

> "continuous replication"

Both can replicate servers.

Instead ask:

> **Why are they replicating the servers?**

```text
To migrate them
→ MGN

To recover them
→ DRS
```

---

# AWS Backup

**AWS Backup = centralized, automated backup and restore.**

It is used to protect supported AWS resources and manage backup policies, schedules, retention, and recovery from a central service. AWS Backup can restore an entire EC2 instance from a recovery point, including its root and data volumes plus supported configuration settings.

## Use it when

> "Create backups and restore resources when needed."

→ **AWS Backup**

### Common keywords

* Backup
* Restore
* Recovery point
* Backup plan
* Retention
* Scheduled backups
* Centralized backup management
* Data protection
* Compliance

### Example

> "The company needs daily backups of its AWS resources and wants centralized backup policies and retention."

→ **AWS Backup**

---

# AWS Backup vs MGN vs DRS

This is an important SAA distinction.

```text
AWS Backup
= protect data/resources

AWS Transform MGN
= migrate servers

AWS DRS
= recover servers after disaster
```

### Think about the action

```text
"Save a copy"
→ AWS Backup

"Move this server to AWS"
→ MGN

"Recover this server when disaster happens"
→ DRS
```

### Comparison

|                                 | AWS Backup           | AWS Transform MGN        | AWS DRS                  |
| ------------------------------- | -------------------- | ------------------------ | ------------------------ |
| Main purpose                    | Backup / restore     | Migration / rehost       | Disaster recovery        |
| Think                           | **Protect**          | **Move**                 | **Recover**              |
| Continuous replication          | No — backup-oriented | Yes                      | Yes                      |
| Whole physical server migration | No                   | **Yes**                  | Not the primary use case |
| Minimal-downtime migration      | No                   | **Yes**                  | Not its primary purpose  |
| Restore after failure           | **Yes**              | Possible but not primary | **Yes**                  |
| Typical keyword                 | Backup / retention   | Lift-and-shift           | Disaster / DR            |

### Critical exam distinction

> **AWS Backup is not the answer just because data needs to be copied.**

If the requirement is:

```text
Copy server
+ OS
+ applications
+ data
+ minimal downtime
+ move to AWS
```

→ **AWS Transform MGN**

If the requirement is:

```text
Create recovery points
+ retention
+ restore later
```

→ **AWS Backup**

---

# AWS DMS + SCT

## AWS Database Migration Service (DMS)

**AWS Database Migration Service (DMS) = move or replicate database data.**

It can perform:

* Full database migration
* Ongoing replication
* Change Data Capture (CDC)
* Database consolidation
* Homogeneous and heterogeneous migrations

### CDC — Change Data Capture

**CDC = continuously capture changes made to the source database and replicate them to the target.**

```text
Full Load
= copy existing data

CDC
= copy ongoing changes
```

### Example

```text
Source database
      ↓
DMS
      ↓
Target database
```

### Memory

> **DMS = move data**

---

# AWS Schema Conversion Tool (SCT)

**AWS Schema Conversion Tool (SCT) = convert database schema and database code when changing database engines.**

Example:

```text
Oracle
   ↓
AWS SCT
= convert schema / database code
   ↓
AWS DMS
= migrate data
   ↓
Aurora PostgreSQL
```

### Memory

> **SCT = convert schema**

---

# DMS vs SCT

```text
DMS
= move data

SCT
= convert schema / database code
```

### Same or compatible database engine

```text
Same / compatible engine
→ DMS
```

### Different database engine

```text
Different engine
→ SCT + DMS
```

### Example

> "Migrate an Oracle database to Amazon Aurora PostgreSQL."

→ **AWS Schema Conversion Tool (SCT) + AWS Database Migration Service (DMS)**

Why?

```text
Oracle
 ↓
SCT
= convert schema / code
 ↓
DMS
= migrate data
 ↓
Aurora PostgreSQL
```

---

# The 7 Rs of Migration

The **7 Rs** describe different migration strategies.

| R                           | Meaning                    | Simple idea                          |
| --------------------------- | -------------------------- | ------------------------------------ |
| **Rehost**                  | Move without major changes | Lift-and-shift                       |
| **Replatform**              | Move with small changes    | Use a managed AWS service            |
| **Repurchase**              | Replace the application    | Buy/use a SaaS product               |
| **Refactor / Re-architect** | Redesign the application   | Build for cloud-native architecture  |
| **Relocate**                | Move the whole environment | Move VMware environment, for example |
| **Retain**                  | Keep it where it is        | Don't migrate yet                    |
| **Retire**                  | Stop using it              | Decommission it                      |

---

## Rehost

**Move without major application changes.**

Example:

```text
On-prem VM
   ↓
EC2
```

Typical signal:

> **Lift-and-shift**

### Associated service

```text
Rehost
→ AWS Transform MGN
```

---

## Replatform

**Move with relatively small changes while taking advantage of managed AWS services.**

Example:

```text
MySQL on EC2
   ↓
Amazon RDS for MySQL
```

The application is not completely redesigned, but the platform changes.

---

## Repurchase

**Replace the existing application with another product or SaaS solution.**

Example:

```text
Self-hosted CRM
   ↓
SaaS CRM
```

You effectively stop using the old application.

---

## Refactor / Re-architect

**Redesign the application to use a different architecture, often cloud-native.**

Example:

```text
Monolith
   ↓
Microservices / serverless architecture
```

Usually involves significant application changes.

---

## Relocate

**Move the entire environment without individually redesigning workloads.**

Example:

```text
VMware environment
   ↓
VMware Cloud on AWS
```

The whole environment is moved rather than converting each workload separately.

---

## Retain

**Keep the workload where it is for now.**

Possible reasons:

* Business requirements
* Technical dependencies
* Compliance constraints
* Migration not currently justified

---

## Retire

**Stop using the application.**

Example:

```text
Unused application
   ↓
Decommission
```

There is no reason to migrate something that is no longer needed.

---

# 7 Rs quick memory

```text
Rehost
= move as-is

Replatform
= move + small changes

Repurchase
= replace with another product

Refactor
= redesign

Relocate
= move the whole environment

Retain
= keep it for now

Retire
= stop using it
```

---

# Migration / Messaging / DR Decision Tree

```text
Existing message broker?
        ↓
RabbitMQ / ActiveMQ
        ↓
Amazon MQ
```

```text
Server migration?
        ↓
Rehost / lift-and-shift
        ↓
AWS Transform MGN
```

```text
Backup / retention / restore?
        ↓
AWS Backup
```

```text
Disaster recovery?
        ↓
AWS Elastic Disaster Recovery (DRS)
```

```text
Database migration?
        ↓
   ┌────┴──────────┐
   ↓               ↓
Same engine   Different engine
   ↓               ↓
DMS            SCT + DMS
```

```text
Customer engagement / legacy campaigns?
        ↓
Amazon Pinpoint
```

```text
Current direct messaging?
        ↓
SMS / voice / push / OTP
        ↓
AWS End User Messaging
```

---

# Common Question Patterns

> **"Existing on-premises application uses RabbitMQ and must migrate with minimal code changes."**

→ **Amazon MQ**

---

> **"Move existing physical servers to AWS with minimal application changes and minimal downtime."**

→ **AWS Transform MGN**

---

> **"A company wants centralized daily backups, retention policies, and recovery points for AWS resources."**

→ **AWS Backup**

---

> **"A company needs disaster recovery for physical, virtual, or cloud servers."**

→ **AWS Elastic Disaster Recovery (DRS)**

---

> **"A company needs a targeted customer campaign using customer segments and journeys."**

→ **Amazon Pinpoint**
**Legacy/current-status note:** Pinpoint support ends **October 30, 2026**.

---

> **"A company needs to send SMS, voice, push, or OTP messages directly from an application."**

→ **AWS End User Messaging**

---

> **"Migrate Oracle to Aurora PostgreSQL."**

→ **AWS Schema Conversion Tool (SCT) + AWS Database Migration Service (DMS)**

---

> **"Continuously replicate database changes while migrating."**

→ **AWS DMS with CDC**

---

# SAA Traps & Distinctions

## Amazon MQ vs SQS / SNS / EventBridge

```text
Existing traditional broker
+ RabbitMQ / ActiveMQ
+ minimal code changes
→ Amazon MQ
```

Whereas:

```text
New AWS-native application
→ SQS / SNS / EventBridge
```

The key question is whether the application **already depends on a traditional message broker**.

---

## Pinpoint vs SNS vs End User Messaging

```text
Legacy targeted customer engagement
→ Amazon Pinpoint

General pub/sub notifications
→ Amazon SNS

Current direct SMS / voice / push / OTP messaging
→ AWS End User Messaging
```

---

## MGN vs DRS

Do not choose based only on replication.

```text
"Move / migrate to AWS"
→ AWS Transform MGN

"Recover after disaster"
→ AWS Elastic Disaster Recovery (DRS)
```

---

## MGN vs AWS Backup

```text
"Create backups and restore later"
→ AWS Backup

"Move running servers to AWS"
→ AWS Transform MGN
```

### Especially important

If the question mentions:

* Physical server
* Operating system
* Applications
* Data
* Minimal downtime
* Lift-and-shift

→ **MGN**

---

## DMS vs SCT

Do not confuse **data migration** with **schema conversion**.

```text
DMS
= move / replicate data

SCT
= convert schema / database code
```

For a cross-engine migration:

```text
SCT + DMS
```

---

# Pocket Card

| Keyword                                         | Answer                                             |
| ----------------------------------------------- | -------------------------------------------------- |
| Existing RabbitMQ / ActiveMQ application        | **Amazon MQ**                                      |
| Existing message broker / minimal code changes  | **Amazon MQ**                                      |
| Legacy targeted customer engagement / campaigns | **Amazon Pinpoint**                                |
| Current SMS / MMS / voice messaging             | **AWS End User Messaging SMS**                     |
| Current push notifications                      | **AWS End User Messaging Push**                    |
| OTP / verification messaging                    | **AWS End User Messaging Notify**                  |
| Minimal-change server migration / rehost        | **AWS Transform MGN**                              |
| Lift-and-shift servers                          | **AWS Transform MGN**                              |
| Physical server → AWS                           | **AWS Transform MGN**                              |
| Backup / retention / restore                    | **AWS Backup**                                     |
| Centralized backup management                   | **AWS Backup**                                     |
| Disaster recovery for servers                   | **AWS Elastic Disaster Recovery (DRS)**            |
| Move database data                              | **AWS DMS**                                        |
| Database replication / migration                | **AWS DMS**                                        |
| Full load + ongoing database changes            | **DMS Full Load + CDC**                            |
| CDC                                             | **Capture and replicate ongoing database changes** |
| Convert database schema                         | **AWS SCT**                                        |
| Different database engines                      | **AWS SCT + DMS**                                  |
| Oracle → Aurora PostgreSQL                      | **AWS SCT + DMS**                                  |

