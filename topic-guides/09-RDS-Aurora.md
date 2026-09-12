# Section 9: RDS & Aurora

## The idea

Running your own database server means patching the OS, applying engine updates, taking backups at 2 a.m., and rebuilding everything when the hardware dies. **RDS (Relational Database Service) = AWS runs the database for you.** You pick an engine — **MySQL, PostgreSQL, MariaDB, Oracle, or SQL Server** — and AWS handles the machinery underneath.

The price of that convenience: **you get NO OS access**. You can't SSH in, can't install custom extensions at the OS level, can't tweak the engine binaries. If a question demands **custom engine configuration or OS-level access**, RDS is out — the answer is **database on EC2** (full control, all the toil) or **RDS Custom** (a halfway house for Oracle/SQL Server).

Now for **THE core distinction — the most-tested database fact on the entire exam.** RDS has two features that both involve "extra copies of your database," and they exist for completely different reasons. Think of your car:

* **Multi-AZ is the spare tire.** It exists for the day something goes wrong. You never drive on it during normal life.
* **Read Replicas are extra checkout lanes at the supermarket.** They exist to serve more customers at once, every ordinary day.

|                       | **Multi-AZ** (availability)                         | **Read Replicas** (performance)                        |
| --------------------- | --------------------------------------------------- | ------------------------------------------------------ |
| Replication           | **Synchronous** (standby is always exactly current) | **Asynchronous** (slight lag)                          |
| Where                 | Standby in **another AZ**                           | **Same AZ, cross-AZ, or cross-REGION**                 |
| How many              | 1 standby                                           | **Up to 15**                                           |
| Can you read from it? | **NO — ZERO reads from the standby**                | **Yes — that's the whole point** (read-only endpoints) |
| Failover              | **Automatic** via failover to standby               | **None automatic — manual promotion only**             |
| Solves                | AZ failure, hardware death                          | Read-heavy workloads, reporting                        |

**THE trap:** *"use the Multi-AZ standby to serve read traffic"* → **impossible**. The standby is invisible until failover.

**THE trap (mirror image):** *"read replica provides automatic failover"* → **no**. Promotion is a **manual** decision — which also makes a cross-region replica a useful DR option, just not an automatic failover mechanism.

And note: **production uses BOTH** — Multi-AZ for surviving failures, replicas for scaling reads. They're not competitors.

## Backups & encryption

* **Automated backups** → enable **PITR (Point-In-Time Recovery)**, retention **maximum 35 days**. Deleted when the instance is deleted.
* **Manual snapshots** → kept **until you delete them**. **"Retain backups for 5 years / 7 years / compliance" → manual snapshot**, always — 35 days is the wall automated backups can't cross.

**Encryption is a birth decision.** You can only encrypt an RDS database **at creation**. To encrypt an existing unencrypted database, do the snapshot dance:

```text
unencrypted DB → snapshot → COPY the snapshot (enable encryption) → restore from encrypted copy
```

You cannot flip encryption on in place, and you cannot encrypt the original snapshot directly — you encrypt the **copy**.

## RDS Proxy

Databases hate being swarmed. Every connection costs memory, and **Lambda** can scale to thousands of concurrent executions and open many database connections. **RDS Proxy = a connection pool** that sits in front of RDS, letting many clients share a smaller set of database connections.

* **Lambda + RDS + too many connections = RDS Proxy.**
* The proxy can also reduce the impact of database failover by keeping client connections and reconnecting to the new database instance.

Also worth one neuron: **storage autoscaling** — RDS can grow its storage automatically when it runs low ("database keeps running out of disk with unpredictable growth" → enable storage autoscaling).

## Oracle on RDS

Amazon RDS supports **Oracle Database**, so you can keep Oracle as the database engine while AWS manages the database infrastructure.

```text
On-premises Oracle
        ↓
      AWS DMS
        ↓
  Amazon RDS for Oracle
```

### Oracle migration — DMS vs SCT vs RMAN

These three are easy to confuse, so keep their jobs separate:

| Tool                                     | What it does                                       |
| ---------------------------------------- | -------------------------------------------------- |
| **AWS DMS (Database Migration Service)** | **Moves / replicates database data**               |
| **AWS SCT (Schema Conversion Tool)**     | **Converts schema when changing database engines** |
| **Oracle RMAN (Recovery Manager)**       | **Oracle backup and recovery**                     |

### AWS DMS

Use **AWS DMS** when moving database data.

Example:

```text
Oracle
  ↓ DMS
RDS for Oracle
```

The database engine stays **Oracle**.

> *"Migrate an on-premises Oracle database to RDS for Oracle"* → **AWS DMS**

### AWS SCT

Use **AWS SCT** when changing the database engine.

Example:

```text
Oracle
  ↓ SCT
PostgreSQL
```

SCT converts the schema and related database code where supported. DMS then moves the actual data.

```text
Oracle
   ↓
SCT → convert schema
   ↓
DMS → migrate data
   ↓
PostgreSQL
```

> **Same engine → DMS**
> **Different engine → SCT + DMS**

### RMAN

**RMAN (Recovery Manager)** is Oracle's backup and recovery tool.

Use it for:

* Oracle database backups
* Oracle database recovery
* restoring Oracle databases

Do not confuse it with migration or high availability:

```text
DMS
= migration

SCT
= schema conversion

RMAN
= backup / recovery

Multi-AZ
= high availability / failover
```

### Oracle Multi-AZ

If the Oracle database must remain available when the primary database or AZ fails:

→ **RDS for Oracle Multi-AZ**

```text
        RDS for Oracle
         /          \
        ↓            ↓
    Primary       Standby
      AZ-A           AZ-B
```

> *"Oracle database must survive an AZ failure with automatic failover"* → **RDS for Oracle Multi-AZ**

### Oracle licensing

Two important choices:

* **License Included** → AWS provides the Oracle license as part of the supported RDS pricing model.
* **BYOL (Bring Your Own License)** → use eligible Oracle licenses you already own.

> *"The company already owns eligible Oracle licenses"* → **BYOL**

### Oracle control

RDS for Oracle is managed, so you do **not** get full operating-system access.

If the question requires:

* OS-level access
* full control over the Oracle server
* custom software that requires server-level access

→ **Oracle on EC2** or **RDS Custom for Oracle**, depending on the requirement.

### Oracle exam rule

```text
Oracle → RDS for Oracle
→ DMS

Oracle → different database engine
→ SCT + DMS

Oracle backup/recovery
→ RMAN

Oracle high availability
→ RDS for Oracle Multi-AZ

Oracle needs OS-level control
→ EC2 / RDS Custom
```

## Aurora

Aurora is AWS's own cloud-native engine, **compatible with MySQL and PostgreSQL** (your app connects the same way). The architectural trick: Aurora **separates compute from storage**.

```text
   [Writer node]  [Reader]  [Reader] ...   ← compute: disposable, pluggable
        │            │         │
   ═════╧════════════╧═════════╧═════════
     Shared storage: 6 copies across 3 AZs, self-healing,
     auto-grows to 128 TB
```

Because every node plugs into the **same shared storage**, a failed writer is replaced by simply promoting a reader that already sees all the data:

* **Fast failover**
* **Up to 15 replicas with low replication lag**
* **Storage auto-grows** — no manual storage provisioning
* **Two endpoints**: the **writer endpoint** points to the current writer and survives failover; the **reader endpoint** distributes reads across Aurora readers.

Apps send writes to the writer endpoint and reads to the reader endpoint — never hardcode instance addresses.

**Aurora's special features — feature → scenario:**

| Feature             | The scenario it answers                                                                                         |
| ------------------- | --------------------------------------------------------------------------------------------------------------- |
| **Serverless v2**   | **Spiky, unpredictable, or idle** workloads — dev/test databases used a few hours a day, capacity scales itself |
| **Global Database** | **Cross-region DR** plus low-latency reads in other Regions                                                     |
| **Cloning**         | **Copy-on-write copy** of a production database for staging/testing                                             |
| **Backtrack**       | **Rewind the database** (Aurora MySQL) after an accidental change, without restoring a new database             |

RPO (Recovery Point Objective) = how much data you may lose; RTO (Recovery Time Objective) = how long until you're back.

## Question patterns

> *"Database must survive an AZ failure with no manual intervention"* → **Multi-AZ**

> *"Reporting/analytics queries are slowing down the production database"* → **Read Replica** (offload reads — but if the queries are the *same ones repeatedly*, **ElastiCache** may be better)

> *"Lambda functions are exhausting database connections"* → **RDS Proxy**

> *"Migrate an Oracle database to RDS for Oracle"* → **AWS DMS**

> *"Convert Oracle to PostgreSQL"* → **AWS SCT + DMS**

> *"Perform Oracle database backup/recovery"* → **RMAN**

> *"Oracle database must remain available after an AZ failure"* → **RDS for Oracle Multi-AZ**

> *"The company already owns eligible Oracle licenses"* → **RDS for Oracle BYOL**

> *"Encrypt an existing unencrypted RDS database"* → **snapshot → copy with encryption → restore**

> *"Global application needs cross-region DR"* → **Aurora Global Database**

> *"Dev database sits idle nights and weekends; minimize cost"* → **Aurora Serverless v2**

> *"Need a full copy of the production database for testing, quickly and cheaply"* → **Aurora Cloning**

> *"Developer ran a bad DELETE; restore the database to an earlier point as fast as possible"* → **Aurora Backtrack**

> *"Compliance requires database backups kept for years"* → **Manual snapshots**

> *"Application needs OS-level access / a custom database engine"* → **EC2** (or **RDS Custom for Oracle/SQL Server**)

## Pocket card

| Keyword in question                         | Answer                                  |
| ------------------------------------------- | --------------------------------------- |
| survive AZ failure, auto-failover           | **Multi-AZ**                            |
| standby serves reads                        | **TRAP — never**                        |
| scale reads, offload reporting              | **Read Replica**                        |
| replica auto-failover                       | **TRAP — promotion is manual**          |
| same queries over and over                  | **ElastiCache**                         |
| Lambda + RDS connections                    | **RDS Proxy**                           |
| backups > 35 days / retain for years        | **Manual snapshot**                     |
| restore to any point in time ≤35 days       | **Automated backups / PITR**            |
| encrypt existing DB                         | **Snapshot → copy encrypted → restore** |
| Oracle → RDS for Oracle                     | **DMS**                                 |
| Oracle → different database engine          | **SCT + DMS**                           |
| Oracle schema conversion                    | **SCT**                                 |
| Oracle backup/recovery                      | **RMAN**                                |
| Oracle high availability                    | **RDS for Oracle Multi-AZ**             |
| Oracle licenses already owned               | **BYOL**                                |
| Oracle needs OS-level control               | **EC2 / RDS Custom**                    |
| OS access / custom engine                   | **EC2 or RDS Custom**                   |
| Aurora MySQL/PostgreSQL-compatible database | **Aurora**                              |
| spiky / idle / unpredictable load           | **Aurora Serverless v2**                |
| cross-region DR / low-latency global reads  | **Aurora Global Database**              |
| instant prod copy for staging               | **Aurora Cloning**                      |
| undo mistake fast                           | **Aurora Backtrack**                    |

You now have the main RDS decisions plus the Oracle-specific migration, backup, licensing, and high-availability patterns in one place.
