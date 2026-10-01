# Section 37A: Migration, Messaging & Disaster Recovery

## The idea

These AWS services commonly appear in SAA questions for **migration, messaging, database migration, and disaster recovery**.

Best approach:

> **Read the requirement → identify the unique keyword → choose the service.**

```text
Existing RabbitMQ application
→ Amazon MQ

Lift-and-shift servers to AWS
→ AWS Application Migration Service (MGN)

Disaster recovery for servers
→ AWS Elastic Disaster Recovery (DRS)

Migrate database data
→ AWS Database Migration Service (DMS)

Different database engines
→ AWS Schema Conversion Tool (SCT) + DMS
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

### Important distinction

```text
Existing broker-based application
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

# AWS Application Migration Service (MGN)

**AWS Application Migration Service (MGN) = rehost / lift-and-shift servers to AWS.**

MGN continuously replicates the source server's **block-level data** to AWS.

```text
On-premises server
        ↓
AWS Replication Agent
        ↓
Continuous replication
        ↓
AWS staging area
        ↓
EC2
```

The goal is to move the server to AWS with **minimal changes**.

## Use it when

> "Move existing physical or virtual servers to AWS without redesigning the application."

→ **AWS Application Migration Service (MGN)**

MGN provides continuous data protection with **recovery points near seconds** and can achieve **recovery in minutes** in appropriate configurations.

## Important terminology

MGN is associated with:

* Rehost
* Lift-and-shift
* Minimal application changes
* Existing physical servers
* Existing virtual servers
* Continuous block-level replication

### Remember

> **Rehost / lift-and-shift → AWS Application Migration Service (MGN)**

### Example

```text
Existing on-premises application
        ↓
No major redesign
        ↓
Move server to AWS
        ↓
AWS Application Migration Service (MGN)
```

### Memory

> **MGN = move servers**

---

# AWS Elastic Disaster Recovery (DRS)

**AWS Elastic Disaster Recovery (DRS) = disaster recovery for servers.**

It continuously replicates workloads into AWS, but the recovery environment is mainly used **when a disaster happens**.

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

> "We need a cost-effective disaster recovery solution for physical, virtual, or cloud servers."

→ **AWS Elastic Disaster Recovery (DRS)**

DRS is designed for **disaster recovery**, rather than simply being a migration mechanism.

### Memory

```text
DRS
= disaster recovery
= recover servers in AWS
```

---

# MGN vs DRS

Both services use continuous replication, so SAA questions can make them look similar.

The key difference is the **goal**:

```text
AWS Application Migration Service (MGN)
= migrate to AWS

AWS Elastic Disaster Recovery (DRS)
= recover in AWS when disaster happens
```

### Mental model

```text
Need to MOVE the workload
→ AWS Application Migration Service (MGN)

Need to RECOVER the workload after a disaster
→ AWS Elastic Disaster Recovery (DRS)
```

### Comparison

| Service                                     | Main purpose                    | Typical signal                   |
| ------------------------------------------- | ------------------------------- | -------------------------------- |
| **AWS Application Migration Service (MGN)** | Rehost / migrate servers to AWS | Lift-and-shift / minimal changes |
| **AWS Elastic Disaster Recovery (DRS)**     | Disaster recovery for servers   | Recover after a disaster         |

### Important distinction

> **MGN and DRS both replicate servers, but the intended outcome is different.**

```text
Migration project
→ MGN

Business continuity / disaster recovery
→ DRS
```

---

# AWS DMS + SCT

## AWS Database Migration Service (DMS)

**AWS Database Migration Service (DMS) = move or replicate database data.**

It is used to migrate data between databases and can also support ongoing replication during migration.

### Memory

> **DMS = move data**

---

## AWS Schema Conversion Tool (SCT)

**AWS Schema Conversion Tool (SCT) = convert schema/code when changing database engines.**

Example:

```text
Oracle
   ↓
AWS SCT → convert schema
   ↓
AWS DMS → move data
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
= convert schema
```

### Same database engine

When source and target use the same or compatible database engine:

```text
Same database engine
→ AWS Database Migration Service (DMS)
```

### Different database engine

When changing database engines:

```text
Different database engine
→ AWS Schema Conversion Tool (SCT) + DMS
```

### Important pattern

```text
Same database engine
→ DMS

Different database engine
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
= convert schema / database code
 ↓
DMS
= migrate the data
 ↓
Aurora PostgreSQL
```

---

# The 7 Rs of Migration

The **7 Rs** describe different migration strategies.

| R                           | Meaning                    | Simple idea                         |
| --------------------------- | -------------------------- | ----------------------------------- |
| **Rehost**                  | Move without major changes | Lift-and-shift                      |
| **Replatform**              | Move with small changes    | Use a managed AWS service           |
| **Repurchase**              | Replace the application    | Buy/use a SaaS product              |
| **Refactor / Re-architect** | Redesign the application   | Build for cloud-native architecture |
| **Relocate**                | Move the whole environment | VMware Cloud on AWS, for example    |
| **Retain**                  | Keep it where it is        | Don't migrate yet                   |
| **Retire**                  | Stop using it              | Decommission it                     |

---

## Rehost

**Move without major changes.**

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
→ AWS Application Migration Service (MGN)
```

---

## Replatform

**Move with small changes.**

Move the workload while taking advantage of a managed AWS service.

Example:

```text
MySQL on EC2
   ↓
Amazon RDS
```

The application architecture is not completely redesigned, but the underlying platform changes.

---

## Repurchase

**Replace the existing application with another product or SaaS solution.**

Example:

```text
Self-hosted CRM
   ↓
SaaS CRM
```

You effectively abandon the old application and use the new one.

---

## Refactor / Re-architect

**Redesign the application to use cloud-native architecture.**

Example:

```text
Monolith
   ↓
Lambda / containers / serverless architecture
```

Usually involves significant application changes.

---

## Relocate

**Move the whole environment without redesigning individual workloads.**

Example:

```text
VMware environment
   ↓
VMware Cloud on AWS
```

The idea is to move the environment rather than individually redesigning each workload.

---

## Retain

**Keep the workload where it is for now.**

Example:

```text
On-premises
   ↓
Keep it there
```

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

# Migration Service Decision Tree

```text
Existing message broker?
        ↓
RabbitMQ / ActiveMQ
        ↓
Amazon MQ
```

```text
Existing server migration?
        ↓
Rehost / lift-and-shift
        ↓
AWS Application Migration Service (MGN)
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

---

# Common Question Patterns

> **"Existing on-premises application uses RabbitMQ and must migrate with minimal code changes."**

→ **Amazon MQ**

---

> **"Move existing servers to AWS with minimal application changes."**

→ **AWS Application Migration Service (MGN)**

---

> **"Need disaster recovery for physical, virtual, or cloud servers."**

→ **AWS Elastic Disaster Recovery (DRS)**

---

> **"Migrate Oracle to Aurora PostgreSQL."**

→ **AWS Schema Conversion Tool (SCT) + AWS Database Migration Service (DMS)**

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

## MGN vs DRS

Do not choose based only on the fact that both replicate servers.

Look at the objective:

```text
"Move / migrate to AWS"
→ AWS Application Migration Service (MGN)

"Recover after disaster"
→ AWS Elastic Disaster Recovery (DRS)
```

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

| Keyword                                        | Answer                                                                      |
| ---------------------------------------------- | --------------------------------------------------------------------------- |
| Existing RabbitMQ / ActiveMQ application       | **Amazon MQ**                                                               |
| Existing message broker / minimal code changes | **Amazon MQ**                                                               |
| Minimal-change server migration / rehost       | **AWS Application Migration Service (MGN)**                                 |
| Lift-and-shift servers                         | **AWS Application Migration Service (MGN)**                                 |
| Disaster recovery for servers                  | **AWS Elastic Disaster Recovery (DRS)**                                     |
| Move database data                             | **AWS Database Migration Service (DMS)**                                    |
| Database replication / migration               | **AWS Database Migration Service (DMS)**                                    |
| Different database engines                     | **AWS Schema Conversion Tool (SCT) + DMS**                                  |
| Convert database schema                        | **AWS Schema Conversion Tool (SCT)**                                        |
| Oracle → Aurora PostgreSQL                     | **AWS Schema Conversion Tool (SCT) + AWS Database Migration Service (DMS)** |

---


