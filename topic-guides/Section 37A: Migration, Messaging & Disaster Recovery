# Section 37A: Migration, Messaging & Disaster Recovery

## The idea

These are AWS services that commonly appear in SAA questions as **specific migration, messaging, database migration, or disaster-recovery use cases**.

You generally don't need deep knowledge of each one.

The best strategy is:

> **Read the requirement → identify the unique keyword → choose the service.**

For example:

```text
Existing RabbitMQ application
→ Amazon MQ

Lift-and-shift servers to AWS
→ AWS Application Migration Service (MGN)

Disaster recovery for servers
→ AWS Elastic Disaster Recovery (DRS)

Migrate database data
→ AWS DMS

Different database engines
→ AWS SCT + DMS
```

---

# Amazon MQ

**Amazon MQ = managed message broker for applications that already use traditional messaging systems.**

It supports managed brokers such as:

* ActiveMQ
* RabbitMQ

It is useful when an existing application already uses traditional messaging protocols or APIs and you want to migrate it to AWS without rewriting the application.

Common keywords:

* RabbitMQ
* ActiveMQ
* JMS
* AMQP
* existing message broker
* minimal application changes

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

The service continuously replicates the source server's **block-level data** to AWS.

```text
On-premises server
        ↓
    MGN agent
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

→ **MGN**

MGN provides continuous data protection with recovery points near seconds and can achieve recovery in minutes in appropriate configurations.

## Important terminology

MGN is associated with:

* Rehost
* Lift-and-shift
* Minimal application changes
* Existing physical servers
* Existing virtual servers
* Continuous block-level replication

### Remember

> **Rehost / lift-and-shift → MGN**

### Example

```text
Existing on-premises application
        ↓
No major redesign
        ↓
Move server to AWS
        ↓
MGN
```

### Memory

> **MGN = MOVE SERVERS**

---

# AWS Elastic Disaster Recovery (DRS)

**AWS Elastic Disaster Recovery = disaster recovery for servers.**

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

→ **AWS DRS**

DRS is designed to provide disaster recovery rather than simply being a migration mechanism.

### Memory

```text
DRS
= disaster recovery
= recover servers in AWS
```

---

# MGN vs DRS

Both services use continuous replication, so they can look very similar in SAA questions.

The key difference is the **goal**.

```text
MGN
= migrate to AWS

DRS
= recover in AWS when disaster happens
```

### Mental model

```text
Need to MOVE the workload
→ MGN

Need to RECOVER the workload after a disaster
→ DRS
```

### Comparison

| Service     | Main purpose                    | Typical signal                   |
| ----------- | ------------------------------- | -------------------------------- |
| **AWS MGN** | Rehost / migrate servers to AWS | Lift-and-shift / minimal changes |
| **AWS DRS** | Disaster recovery for servers   | Recover after a disaster         |

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

For example:

```text
Oracle
   ↓
SCT → convert schema
   ↓
DMS → move data
   ↓
Aurora PostgreSQL
```

### Memory

> **SCT = convert schema**

---

# DMS vs SCT

The easiest way to remember them:

```text
DMS
= MOVE DATA

SCT
= CONVERT SCHEMA
```

### Same database engine

When the source and target use the same or compatible database engine:

```text
Same database engine
→ DMS
```

### Different database engine

When changing database engines:

```text
Different database engine
→ SCT + DMS
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

→ **AWS SCT + DMS**

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

| R                           | Meaning                    | Simple idea                            |
| --------------------------- | -------------------------- | -------------------------------------- |
| **Rehost**                  | Move without major changes | Lift and shift                         |
| **Replatform**              | Move with small changes    | Use a managed AWS service              |
| **Repurchase**              | Replace the application    | Buy a SaaS product                     |
| **Refactor / Re-architect** | Redesign the application   | Build it for cloud-native architecture |
| **Relocate**                | Move the whole environment | VMware Cloud on AWS, for example       |
| **Retain**                  | Keep it where it is        | Don't migrate yet                      |
| **Retire**                  | Stop using it              | Decommission it                        |

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
→ MGN
```

---

## Replatform

**Move with small changes.**

You move the workload while taking advantage of a managed AWS service.

Example:

```text
MySQL on EC2
   ↓
Amazon RDS
```

The application architecture is not completely redesigned, but the underlying platform changes.

---

## Repurchase

**Replace the existing application with a different product or SaaS solution.**

Example:

```text
Self-hosted CRM
   ↓
SaaS CRM
```

You effectively abandon the old application and purchase/use a new one.

---

## Refactor / Re-architect

**Redesign the application to take advantage of cloud-native architecture.**

Example:

```text
Monolith
   ↓
Lambda / containers / serverless architecture
```

This usually involves significant application changes.

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

Reasons can include:

* business requirements
* technical dependencies
* compliance constraints
* migration not currently justified

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

A useful SAA decision process is:

```text
What is the question asking?

          ┌───────────────────────────┐
          │ Existing message broker?  │
          └─────────────┬─────────────┘
                        ↓
                 RabbitMQ / ActiveMQ
                        ↓
                   Amazon MQ
```

```text
          ┌───────────────────────────┐
          │ Existing server migration?│
          └─────────────┬─────────────┘
                        ↓
              Rehost / lift-and-shift
                        ↓
                     MGN
```

```text
          ┌───────────────────────────┐
          │ Disaster recovery?        │
          └─────────────┬─────────────┘
                        ↓
                       DRS
```

```text
          ┌───────────────────────────┐
          │ Database migration?       │
          └─────────────┬─────────────┘
                        ↓
             ┌──────────┴──────────┐
             ↓                     ↓
       Same engine           Different engine
             ↓                     ↓
            DMS                SCT + DMS
```

---

# Common Question Patterns

> **"Existing on-premises application uses RabbitMQ and must migrate with minimal code changes."**

→ **Amazon MQ**

---

> **"Move existing servers to AWS with minimal application changes."**

→ **AWS Application Migration Service (MGN)**

---

> **"Need disaster recovery for physical/virtual/cloud servers."**

→ **AWS Elastic Disaster Recovery (DRS)**

---

> **"Migrate Oracle to Aurora PostgreSQL."**

→ **AWS SCT + DMS**

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
→ MGN

"Recover after disaster"
→ DRS
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

| Keyword                                        | Answer                                  |
| ---------------------------------------------- | --------------------------------------- |
| Existing RabbitMQ / ActiveMQ application       | **Amazon MQ**                           |
| Existing message broker / minimal code changes | **Amazon MQ**                           |
| Minimal-change server migration / rehost       | **AWS MGN**                             |
| Lift-and-shift servers                         | **AWS MGN**                             |
| Disaster recovery for servers                  | **AWS Elastic Disaster Recovery (DRS)** |
| Move database data                             | **DMS**                                 |
| Database replication / migration               | **DMS**                                 |
| Different database engines                     | **SCT + DMS**                           |
| Convert database schema                        | **SCT**                                 |
| Oracle → Aurora PostgreSQL                     | **SCT + DMS**                           |

---

# Final Memory

```text
Amazon MQ
= EXISTING MESSAGE BROKER
= RABBITMQ / ACTIVEMQ
= MINIMAL CODE CHANGES

MGN
= MOVE SERVERS
= REHOST
= LIFT-AND-SHIFT

DRS
= RECOVER SERVERS
= DISASTER RECOVERY

DMS
= MOVE DATABASE DATA
= DATABASE MIGRATION / REPLICATION

SCT
= CONVERT DATABASE SCHEMA
= DIFFERENT DATABASE ENGINES

7 Rs
= REHOST
= REPLATFORM
= REPURCHASE
= REFACTOR / RE-ARCHITECT
= RELOCATE
= RETAIN
= RETIRE
```

# The Golden Rule

```text
Existing RabbitMQ / ActiveMQ
→ Amazon MQ

Rehost / lift-and-shift
→ MGN

Disaster recovery
→ DRS

Move database data
→ DMS

Different database engines
→ SCT + DMS
```

> **Don't memorize the implementation.**
>
> **Memorize the unique signal.**

For example:

```text
RabbitMQ            → MQ
ActiveMQ            → MQ
Rehost              → MGN
Lift-and-shift      → MGN
Disaster recovery   → DRS
Database migration  → DMS
Different engines   → SCT + DMS
Schema conversion   → SCT
```
