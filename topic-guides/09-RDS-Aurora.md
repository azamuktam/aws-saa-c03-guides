# Section 9: RDS & Aurora

## The idea

**Amazon RDS (Relational Database Service)** is a managed relational database service. AWS manages the underlying infrastructure, operating system maintenance, backups, patching, and database setup.

Supported engines include:

* MySQL
* PostgreSQL
* MariaDB
* Oracle
* SQL Server

RDS does **not** give you normal OS-level access to the database server.

If the question requires:

* Full OS/server control
* Custom database software
* OS-level configuration

→ consider **EC2** or, for supported Oracle/SQL Server scenarios, **RDS Custom**.

---

# Multi-AZ vs Read Replicas

This is one of the most important RDS distinctions for SAA.

|                            | **Multi-AZ**                         | **Read Replica**                                               |
| -------------------------- | ------------------------------------ | -------------------------------------------------------------- |
| Main purpose               | High availability                    | Read scaling                                                   |
| Replication                | Synchronous for standard Multi-AZ    | Asynchronous                                                   |
| Read from standby/replica? | **No**                               | **Yes**                                                        |
| Automatic failover         | **Yes**                              | **No** as a normal read-replica feature                        |
| Location                   | Another AZ / depending on deployment | Same Region, another AZ, or another Region depending on engine |
| Main problem solved        | Infrastructure/AZ failure            | Read-heavy workloads / DR copy                                 |

### Multi-AZ

Multi-AZ creates a standby that can be used automatically if the primary fails.

```text
Primary
   │
   └── synchronous replication ──► Standby
                                      ↓
                                   Failover
```

The standby is **not for normal read traffic**.

### Read Replica

A read replica receives changes asynchronously from the primary.

```text
Primary
   │
   ├──► Read Replica
   ├──► Read Replica
   └──► Read Replica
```

Applications can send read traffic to the replicas.

### Exam patterns

> "Database must survive an AZ failure with automatic failover."

→ **Multi-AZ**

> "Production database is overloaded by reporting queries."

→ **Read Replica**

> "Use the Multi-AZ standby to serve read traffic."

→ **Wrong**

> "Read replica automatically replaces the primary."

→ **Wrong as a normal read-replica feature; promotion is a separate action.**

A real production architecture can use both:

```text
Multi-AZ
→ availability

Read Replicas
→ read scaling
```

---

# Backups and recovery

## Automated backups

Automated backups provide **Point-in-Time Recovery (PITR)**.

They are designed for restoring the database to a specific point within the configured backup-retention period.

For standard RDS DB instances, the retention period is:

**0–35 days**

### Exam signal

> "Restore the database to a specific point in time."

→ **Automated backups / PITR**

---

## Manual snapshots

Manual snapshots remain until you delete them.

They are useful when you need:

* Long-term retention
* Compliance retention
* Backup kept for months or years

### Exam signal

> "Keep a database backup for 10 years."

→ **Manual snapshot**

---

# Encrypting RDS

Encryption should be enabled when the database is created.

For an existing unencrypted RDS DB instance, the usual process is:

```text
Unencrypted DB
      ↓
Create snapshot
      ↓
Copy snapshot with encryption enabled
      ↓
Restore encrypted DB
```

You cannot simply turn on encryption for an existing unencrypted RDS DB instance.

---

# RDS Proxy

**RDS Proxy** is a managed database connection pool.

It is particularly useful when many short-lived applications create large numbers of database connections.

Common example:

```text
Lambda
  ↓
many concurrent connections
  ↓
RDS Proxy
  ↓
RDS
```

RDS Proxy:

* pools database connections
* reduces connection overhead
* helps protect the database from connection storms
* is especially useful with Lambda

### Exam signal

> "Lambda creates too many connections to RDS."

→ **RDS Proxy**

Do not confuse this with read scaling:

```text
RDS Proxy
→ connection management

Read Replica
→ read scaling
```

---

# RDS Storage Auto Scaling

RDS can automatically increase allocated storage when the database approaches its storage threshold.

### Exam signal

> "Database storage is growing unpredictably and administrators don't want to manually increase storage."

→ **RDS Storage Auto Scaling**

---

# Oracle on RDS

Amazon RDS supports Oracle Database.

Example:

```text
On-premises Oracle
        ↓
      AWS DMS
        ↓
  RDS for Oracle
```

RDS for Oracle gives you a managed Oracle database without requiring you to manage the underlying server yourself.

---

## Oracle migration: DMS vs SCT vs RMAN

These are easy to confuse.

| Tool            | Main purpose                                       |
| --------------- | -------------------------------------------------- |
| **AWS DMS**     | Migrate/replicate database data                    |
| **AWS SCT**     | Convert schema/code when changing database engines |
| **Oracle RMAN** | Oracle backup and recovery                         |

### Same engine

```text
Oracle
  ↓
DMS
  ↓
RDS for Oracle
```

> **Oracle → RDS for Oracle** → **DMS**

### Different engine

```text
Oracle
   ↓
SCT
   ↓
PostgreSQL schema/code
   ↓
DMS
   ↓
PostgreSQL
```

> **Oracle → PostgreSQL** → **SCT + DMS**

### RMAN

RMAN is for:

* Oracle backup
* Oracle recovery

Not for general database migration.

### Easy rule

```text
Same engine
→ DMS

Different engine
→ SCT + DMS

Oracle backup/recovery
→ RMAN
```

---

# Oracle High Availability

For an Oracle database that must survive an AZ failure with automatic failover:

→ **RDS for Oracle Multi-AZ**

```text
AZ-A
Primary
  │
  │ synchronous replication
  ▼
AZ-B
Standby
```

### Exam signal

> "Oracle database must remain available after an AZ failure."

→ **RDS for Oracle Multi-AZ**

---

# Oracle licensing

Two important models:

### License Included

AWS provides the Oracle license under the supported RDS licensing model.

### BYOL

**Bring Your Own License**

Use this when the organization already owns eligible Oracle licenses.

### Exam signal

> "The company already owns eligible Oracle licenses."

→ **BYOL**

---

# Oracle and OS control

RDS for Oracle is managed.

If the question requires:

* OS-level access
* Custom Oracle configuration requiring server access
* Full control of the underlying server

→ **EC2** or **RDS Custom for Oracle**, depending on the scenario.

---

# Aurora

**Amazon Aurora** is AWS's managed relational database engine compatible with:

* MySQL
* PostgreSQL

The major architectural difference from standard RDS is Aurora's shared cluster storage.

An Aurora cluster contains:

```text
Aurora Cluster
│
├── Primary / Writer
├── Aurora Replica
├── Aurora Replica
└── Aurora Replica
        │
        ▼
   Shared cluster storage
```

Aurora storage is automatically replicated across multiple Availability Zones.

Aurora supports:

**1 primary + up to 15 Aurora Replicas**

---

# Aurora endpoints

Aurora has several important endpoints.

| Endpoint                      | Purpose                                            |
| ----------------------------- | -------------------------------------------------- |
| **Cluster / Writer endpoint** | Connect to the current primary/writer              |
| **Reader endpoint**           | Distribute read connections across Aurora Replicas |
| **Instance endpoint**         | Connect to one specific DB instance                |
| **Custom endpoint**           | Connect to a selected group of Aurora instances    |

---

## Cluster / Writer endpoint

Use it for normal read/write traffic that should go to the current writer.

The endpoint follows the writer after failover.

### Exam signal

> "Application must always connect to the current Aurora writer."

→ **Cluster / Writer endpoint**

---

## Reader endpoint

The reader endpoint distributes **read connections** among Aurora Replicas.

### Exam signal

> "Distribute read traffic across Aurora Replicas."

→ **Reader endpoint**

Important:

> It balances **connections**, not individual SQL queries.

---

## Instance endpoint

Connects directly to one specific Aurora DB instance.

### Exam signal

> "Connect to a specific Aurora DB instance."

→ **Instance endpoint**

---

## Custom endpoints

Custom endpoints allow you to group selected Aurora DB instances behind a dedicated endpoint.

Example:

```text
Aurora Cluster
│
├── Writer
├── Reader A  ← high capacity
├── Reader B  ← high capacity
├── Reader C  ← low capacity
└── Reader D  ← low capacity
```

You could create:

```text
Production endpoint
→ Reader A + Reader B

Reporting endpoint
→ Reader C + Reader D
```

This allows different workloads to use different groups of Aurora instances.

### Exam signal

> "Production uses high-capacity instances while reporting uses low-capacity instances."

→ **Aurora Custom Endpoint**

---

# Aurora Serverless v2

Aurora Serverless v2 automatically adjusts database capacity as workload changes.

Use it for:

* Unpredictable workloads
* Spiky workloads
* Intermittent workloads
* Applications that don't need fixed capacity all the time

### Exam signal

> "The database has unpredictable traffic and sometimes sits mostly idle."

→ **Aurora Serverless v2**

---

# Aurora Global Database

Aurora Global Database is designed for **cross-Region architectures**.

Use it when you need:

* Cross-Region disaster recovery
* Very low cross-Region replication lag
* Fast recovery after a Regional failure
* Low-latency reads from multiple Regions

Architecture:

```text
Primary Region
      │
      ▼
Aurora Global Database
      │
      ▼
Secondary Region
```

The secondary Region can also serve read traffic.

### Important SAA distinction

```text
RDS Multi-AZ
→ high availability within a Region

Aurora Global Database
→ disaster recovery across Regions
```

### Exam signal

> "Aurora application needs cross-Region DR with very low RPO and rapid recovery."

→ **Aurora Global Database**

---

# RDS cross-Region Read Replicas

Standard RDS engines can also use **cross-Region Read Replicas** for disaster recovery.

Example:

```text
Primary Region
      │
      ▼
RDS PostgreSQL
      │
      │ asynchronous replication
      ▼
Secondary Region
RDS PostgreSQL Read Replica
```

Use this when:

* You need a cross-Region copy of an RDS database.
* The database engine is a supported RDS engine.
* You need disaster recovery.
* You may also use the replica for read scaling.

### Important

Replication is **asynchronous**, so there can be replication lag.

A cross-Region read replica is not equivalent to Multi-AZ:

```text
Multi-AZ
→ regional high availability

Cross-Region Read Replica
→ cross-Region DR / read scaling
```

---

# Cross-Region DR: RDS Read Replica vs Aurora Global Database

Both can provide cross-Region disaster recovery for relational databases, but the exam wording matters.

| Solution                          | Database                | Main use                                                  |
| --------------------------------- | ----------------------- | --------------------------------------------------------- |
| **RDS cross-Region Read Replica** | RDS relational engines  | Cross-Region DR / read scaling                            |
| **Aurora Global Database**        | Aurora MySQL/PostgreSQL | Cross-Region DR + very low replication lag + global reads |

### SAA decision rule

> **Cross-Region DR for an RDS database**
> → **Cross-Region Read Replica**

> **Aurora + very low RPO + very fast cross-Region recovery**
> → **Aurora Global Database**

### Typical exam scenario

> "A relational database requires an RPO of around one second and an RTO of less than one minute after a Regional failure."

→ **Aurora Global Database**

This is stronger than simply choosing a generic cross-Region RDS read replica when the question emphasizes **very low RPO and very fast recovery**.

---

# Don't confuse relational and non-relational global databases

Some SAA questions deliberately put different database types together.

| Service                    | Database type | Typical clue                |
| -------------------------- | ------------- | --------------------------- |
| **RDS**                    | Relational    | SQL database                |
| **Aurora**                 | Relational    | MySQL/PostgreSQL-compatible |
| **DynamoDB Global Tables** | NoSQL         | Multi-Region NoSQL          |
| **Timestream**             | Time-series   | Time-series / IoT / metrics |

### Exam elimination

> "The database must be relational."

Eliminate:

* DynamoDB Global Tables
* Timestream

Then compare:

* RDS cross-Region Read Replica
* Aurora Global Database

---

# Aurora Cloning

Aurora cloning creates a database copy using **copy-on-write**.

It is useful when you need a quick copy of a production database for:

* Testing
* Development
* Staging

### Exam signal

> "Create a fast copy of production for testing without immediately duplicating all the storage."

→ **Aurora Cloning**

---

# Aurora Backtrack

Aurora Backtrack allows an **Aurora MySQL** database to be rewound to an earlier point in time without performing a traditional restore to a new database.

Typical scenario:

```text
10:00 → good data
10:15 → accidental DELETE
10:20 → discover mistake

Backtrack
    ↓
Return database to earlier point
```

### Exam signal

> "An application accidentally changed/deleted data and needs to quickly rewind the Aurora MySQL database."

→ **Aurora Backtrack**

---

# Aurora storage

Aurora uses a distributed cluster volume rather than traditional database-local storage.

The storage is:

* Automatically replicated across multiple AZs
* Self-healing
* Shared by the Aurora DB instances

### Important SAA idea

Aurora readers do **not** need independent full copies of the database.

They use the shared Aurora cluster storage.

This helps Aurora fail over quickly to another DB instance.

---

# Question patterns

> **"Database must survive an AZ failure with automatic failover."**
> → **RDS Multi-AZ**

> **"Reporting queries are consuming too much capacity on the primary."**
> → **Read Replica**

> **"Lambda is exhausting database connections."**
> → **RDS Proxy**

> **"Database storage is growing unpredictably."**
> → **RDS Storage Auto Scaling**

> **"Restore the RDS database to a specific point in time."**
> → **Automated backups / PITR**

> **"Keep RDS backups for years."**
> → **Manual snapshots**

> **"Encrypt an existing unencrypted RDS database."**
> → **Snapshot → copy with encryption → restore**

> **"Migrate Oracle to RDS for Oracle."**
> → **AWS DMS**

> **"Convert Oracle to PostgreSQL."**
> → **AWS SCT + DMS**

> **"Back up or recover an Oracle database."**
> → **RMAN**

> **"Oracle needs automatic failover after an AZ failure."**
> → **RDS for Oracle Multi-AZ**

> **"Company already owns eligible Oracle licenses."**
> → **BYOL**

> **"Application requires OS-level access to the Oracle server."**
> → **EC2 / RDS Custom**

> **"Distribute Aurora read connections across replicas."**
> → **Reader endpoint**

> **"Always connect to the current Aurora writer."**
> → **Cluster / Writer endpoint**

> **"Connect directly to one Aurora instance."**
> → **Instance endpoint**

> **"Production and reporting should use different groups of Aurora instances."**
> → **Custom endpoint**

> **"Aurora workload is unpredictable or intermittent."**
> → **Aurora Serverless v2**

> **"Aurora needs cross-Region disaster recovery and fast recovery."**
> → **Aurora Global Database**

> **"RDS PostgreSQL needs a copy in another Region for DR."**
> → **Cross-Region Read Replica**

> **"Relational database requires very low RPO and very fast cross-Region recovery."**
> → **Aurora Global Database**

> **"Multi-Region NoSQL database."**
> → **DynamoDB Global Tables**

> **"Time-series / IoT metrics database."**
> → **Amazon Timestream**

> **"Quick copy of Aurora production database for testing."**
> → **Aurora Cloning**

> **"Quickly rewind Aurora MySQL after an accidental change."**
> → **Aurora Backtrack**

---

# Pocket card

| Keyword                                          | Answer                                  |
| ------------------------------------------------ | --------------------------------------- |
| AZ failure                                       | **Multi-AZ**                            |
| Automatic regional failover                      | **Multi-AZ**                            |
| Scale database reads                             | **Read Replica**                        |
| Cross-Region RDS DR                              | **Cross-Region Read Replica**           |
| Aurora cross-Region DR                           | **Aurora Global Database**              |
| Very low RPO + fast cross-Region recovery        | **Aurora Global Database**              |
| Lambda + too many DB connections                 | **RDS Proxy**                           |
| Restore to point in time                         | **Automated backup / PITR**             |
| Keep backup for years                            | **Manual snapshot**                     |
| Existing unencrypted RDS → encrypted             | **Snapshot → encrypted copy → restore** |
| Oracle → RDS Oracle                              | **DMS**                                 |
| Oracle → different DB engine                     | **SCT + DMS**                           |
| Oracle backup/recovery                           | **RMAN**                                |
| Oracle AZ HA                                     | **RDS Multi-AZ**                        |
| Existing Oracle license                          | **BYOL**                                |
| Need OS-level DB control                         | **EC2 / RDS Custom**                    |
| Aurora current writer                            | **Writer/Cluster endpoint**             |
| Aurora read balancing                            | **Reader endpoint**                     |
| One specific Aurora instance                     | **Instance endpoint**                   |
| Different workloads → different Aurora instances | **Custom endpoint**                     |
| Spiky/unpredictable Aurora workload              | **Serverless v2**                       |
| Cross-Region Aurora DR                           | **Global Database**                     |
| Quick Aurora copy                                | **Cloning**                             |
| Rewind Aurora MySQL                              | **Backtrack**                           |
| Multi-Region NoSQL                               | **DynamoDB Global Tables**              |
| Time-series database                             | **Timestream**                          |

---

# Core mental model

When an RDS/Aurora question appears, first identify **what problem the question is solving**.

```text
AZ failure
    ↓
Multi-AZ

Need more read capacity
    ↓
Read Replica

Too many application connections
    ↓
RDS Proxy

Cross-Region RDS DR
    ↓
Cross-Region Read Replica

Aurora + cross-Region DR
    ↓
Aurora Global Database

Spiky Aurora capacity
    ↓
Aurora Serverless v2

Different Aurora workloads → different instance groups
    ↓
Custom Endpoint

Quick Aurora copy
    ↓
Aurora Clone

Undo Aurora MySQL changes
    ↓
Backtrack
```

And for Question 39-type elimination:

```text
Relational
   ↓
RDS / Aurora

NoSQL
   ↓
DynamoDB Global Tables

Time-series
   ↓
Timestream
```

The most important SAA distinction in this entire section is:

**Multi-AZ = high availability within a Region.**

**Read Replica = read scaling and can also be used for DR.**

**Aurora Global Database = Aurora cross-Region DR with very low replication lag and fast recovery.**
