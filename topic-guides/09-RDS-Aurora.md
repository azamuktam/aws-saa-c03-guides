# Section 9: RDS & Aurora

## The idea

Running your own database server means patching the OS, applying engine updates, taking backups, and rebuilding everything when the hardware fails. **RDS (Relational Database Service) = AWS runs the database for you.** You pick an engine — **MySQL, PostgreSQL, MariaDB, Oracle, or SQL Server** — and AWS manages the underlying infrastructure.

The price of that convenience: **you get NO OS access**. You can't SSH into the database host or install arbitrary OS-level software. If a question requires **full OS/server control or custom database software**, consider **database on EC2** or, for supported Oracle/SQL Server use cases, **RDS Custom**.

Now for **THE core distinction — one of the most-tested database facts on the exam.** RDS has two features that both involve extra database copies, but they solve different problems.

- **Multi-AZ = high availability.** It provides a standby for failure and automatic failover.
- **Read Replicas = read scaling.** They provide additional read capacity for applications, reporting, and analytics.

| | **Multi-AZ** (availability) | **Read Replicas** (performance) |
|---|---|---|
| Replication | **Synchronous** | **Asynchronous** |
| Main purpose | High availability / failover | Read scaling |
| Where | Standby in **another AZ** | Same AZ, cross-AZ, or cross-Region depending on engine |
| Can you read from it? | **NO — standby is not for normal read traffic** | **Yes** |
| Failover | **Automatic** | **No automatic failover as a normal read replica** — promotion is a separate action |
| Solves | AZ / infrastructure failure | Read-heavy workloads, reporting |

**THE trap:** *"Use the Multi-AZ standby to serve read traffic."* → **No.** A standby is for high availability, not read scaling.

**THE trap:** *"A read replica automatically replaces the primary when it fails."* → **No.** A read replica can be promoted, but it is not the same automatic HA mechanism as Multi-AZ.

And note: **production systems can use BOTH** — Multi-AZ for high availability and read replicas for read scaling.

## Backups & encryption

- **Automated backups** → provide **PITR (Point-In-Time Recovery)**. Retention can be **0–35 days** for standard RDS DB instances; the maximum is **35 days**. 
- **Manual snapshots** → remain until you delete them. **"Keep backups for years / compliance" → manual snapshots.**

**Encryption is a creation-time decision.** To encrypt an existing unencrypted RDS database:

```text
unencrypted DB
      ↓
snapshot
      ↓
copy snapshot with encryption enabled
      ↓
restore DB from encrypted snapshot
```

You cannot simply switch encryption on for an existing unencrypted DB instance.

## RDS Proxy

RDS Proxy is a **managed connection pool** in front of RDS.

This is especially useful when Lambda creates many concurrent connections:

```text
Lambda
  ↓
many connections
  ↓
RDS Proxy
  ↓
RDS
```

RDS Proxy lets many application connections share a smaller number of database connections.

- **Lambda + RDS + too many connections → RDS Proxy.**
- It can also make database failover easier for applications by keeping connections at the proxy layer.

Also worth one neuron: **storage autoscaling** — RDS can automatically increase storage when the database approaches its configured storage limit.

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

| Tool | What it does |
|---|---|
| **AWS DMS (Database Migration Service)** | **Moves / replicates database data** |
| **AWS SCT (Schema Conversion Tool)** | **Converts schema/code when changing database engines** |
| **Oracle RMAN (Recovery Manager)** | **Oracle backup and recovery** |

### AWS DMS

Use **AWS DMS** to migrate or continuously replicate database data.

Example:

```text
Oracle
  ↓ DMS
RDS for Oracle
```

The database engine stays **Oracle**.

> *"Migrate an on-premises Oracle database to RDS for Oracle."* → **AWS DMS**

### AWS SCT

Use **AWS SCT** when changing the database engine.

Example:

```text
Oracle
  ↓ SCT
PostgreSQL
```

SCT converts supported schema and database code. DMS then moves the actual data.

```text
Oracle
   ↓
SCT → convert schema/code
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

- Oracle database backups
- Oracle database recovery
- restoring Oracle databases

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

> *"Oracle database must survive an AZ failure with automatic failover."* → **RDS for Oracle Multi-AZ**

### Oracle licensing

Two important choices:

- **License Included** → AWS provides the Oracle license under the supported RDS licensing model.
- **BYOL (Bring Your Own License)** → use eligible Oracle licenses you already own.

> *"The company already owns eligible Oracle licenses."* → **BYOL**

### Oracle control

RDS for Oracle is managed, so you do **not** get full operating-system access.

If the question requires:

- OS-level access
- full control over the Oracle server
- custom software that requires server-level access

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

Aurora is AWS's managed relational database service, **compatible with MySQL and PostgreSQL**.

The important architectural idea is:

> **Aurora separates database compute from shared cluster storage.**

An Aurora DB cluster contains:

- **1 primary (writer) DB instance**
- **0–15 Aurora Replicas (reader DB instances)**
- **shared cluster storage**

```text
Aurora DB Cluster
│
├── Primary / Writer
│
├── Reader / Aurora Replica
├── Reader / Aurora Replica
└── Reader / Aurora Replica
        │
        ↓
   Shared cluster storage
```

Aurora's cluster storage spans multiple Availability Zones. Aurora can have up to **15 Aurora Replicas** in addition to the primary. 

Because the writer and readers use the same underlying cluster storage, Aurora can fail over to an available reader without copying the whole database.

### Aurora endpoints

Aurora provides different endpoints for different connection patterns:

| Endpoint | What it does |
|---|---|
| **Cluster / Writer endpoint** | Connects to the **current primary/writer**; handles reads and writes |
| **Reader endpoint** | Load-balances **read connections** across Aurora Replicas |
| **Instance endpoint** | Connects to **one specific DB instance** |
| **Custom endpoint** | Connects to a **specific group of Aurora DB instances** |

The built-in reader endpoint balances **connections** among Aurora Replicas; it does not balance individual queries. 

### Aurora Custom Endpoints

A **custom endpoint** lets you group specific Aurora DB instances and give that group its own endpoint.

This is useful when different instances have different capacities or purposes.

Example:

```text
Aurora Cluster
│
├── Writer        - high capacity
├── Reader A      - high capacity
├── Reader B      - high capacity
├── Reader C      - low capacity
└── Reader D      - low capacity
```

Create:

```text
Production endpoint
→ high-capacity instances

Reporting endpoint
→ low-capacity instances
```

Then:

```text
Production application
        ↓
Production custom endpoint
        ↓
High-capacity instances

Internal reporting
        ↓
Reporting custom endpoint
        ↓
Low-capacity instances
```

This is exactly what custom endpoints are designed for: routing different workloads to different subsets of Aurora instances.

A provisioned Aurora cluster can have up to **five custom endpoints**.

### Endpoint exam traps

> *"Send all read traffic to every Aurora Replica."* → **Reader endpoint**

> *"Send reporting traffic only to low-capacity Aurora instances, while production uses high-capacity instances."* → **Custom endpoints**

> *"Connect to one specific Aurora DB instance."* → **Instance endpoint**

> *"Send writes to the current writer, even after failover."* → **Cluster / writer endpoint**

### Aurora availability and replicas

Aurora can automatically fail over to one of the available Aurora Replicas when the primary fails. Aurora Replicas also improve availability and read capacity. 

### Aurora storage

Aurora's cluster volume is replicated across **three Availability Zones** and is self-healing. The current maximum cluster volume is **256 TiB**. 

Do not confuse this with an older **128 TiB** figure found in older study material.

**Aurora's special features — feature → scenario:**

| Feature | The scenario it answers |
|---|---|
| **Serverless v2** | **Spiky, unpredictable, or intermittently used** workloads |
| **Global Database** | **Cross-Region DR** plus low-latency reads in other Regions |
| **Cloning** | **Copy-on-write copy** of a production database for staging/testing |
| **Backtrack** | **Rewind Aurora MySQL** after an accidental change without restoring a separate database |

### Aurora Serverless v2

Use it when database capacity changes significantly over time.

Example:

> "A development database is heavily used during working hours and mostly idle at night."

→ **Aurora Serverless v2**

### Aurora Global Database

Use it for:

- cross-Region disaster recovery
- low-latency reads in other Regions

Aurora Global Database uses storage-based replication with typical cross-Region replication latency of less than one second, and a secondary Region can be promoted in less than one minute in a Regional failure scenario. 

### Aurora Cloning

Aurora cloning uses **copy-on-write** so you can create a database copy for testing or staging without immediately duplicating all underlying data.

> *"Create a quick copy of production for testing."* → **Aurora Cloning**

### Aurora Backtrack

Aurora MySQL **Backtrack** lets you rewind a database to an earlier point in time.

> *"Developer accidentally deleted data and wants to undo the change quickly without restoring a new database."* → **Aurora Backtrack**

## Question patterns

> *"Database must survive an AZ failure with no manual intervention."* → **Multi-AZ**

> *"Reporting/analytics queries are slowing down the production database."* → **Read Replica** (offload reads — but if the same data is repeatedly requested, **ElastiCache** may be better)

> *"Lambda functions are exhausting database connections."* → **RDS Proxy**

> *"Migrate an Oracle database to RDS for Oracle."* → **AWS DMS**

> *"Convert Oracle to PostgreSQL."* → **AWS SCT + DMS**

> *"Perform Oracle database backup/recovery."* → **RMAN**

> *"Oracle database must remain available after an AZ failure."* → **RDS for Oracle Multi-AZ**

> *"The company already owns eligible Oracle licenses."* → **RDS for Oracle BYOL**

> *"Encrypt an existing unencrypted RDS database."* → **snapshot → copy with encryption → restore**

> *"Production traffic should use high-capacity Aurora instances while reporting uses low-capacity instances."* → **Aurora Custom Endpoints**

> *"Read-only traffic should be automatically distributed across Aurora Replicas."* → **Aurora Reader Endpoint**

> *"Connect directly to one specific Aurora instance."* → **Aurora Instance Endpoint**

> *"Writes must always go to whichever Aurora instance is currently the writer."* → **Aurora Cluster/Writer Endpoint**

> *"Global application needs cross-Region DR and low-latency reads."* → **Aurora Global Database**

> *"Development database has unpredictable or intermittent usage."* → **Aurora Serverless v2**

> *"Need a quick copy of production for testing."* → **Aurora Cloning**

> *"Developer accidentally changed/deleted data and wants to rewind Aurora MySQL quickly."* → **Aurora Backtrack**

> *"Compliance requires database backups kept for years."* → **Manual snapshots**

> *"Application needs OS-level access / a custom database engine."* → **EC2** (or **RDS Custom for Oracle/SQL Server**)

## Pocket card

| Keyword in question | Answer |
|---|---|
| survive AZ failure, auto-failover | **Multi-AZ** |
| standby serves reads | **TRAP — no** |
| scale reads, offload reporting | **Read Replica** |
| replica automatically fails over | **TRAP — promotion is separate** |
| same data queried repeatedly | **ElastiCache** |
| Lambda + RDS connections | **RDS Proxy** |
| backups > 35 days / retain for years | **Manual snapshot** |
| restore to a point in time | **Automated backups / PITR** |
| encrypt existing DB | **Snapshot → copy encrypted → restore** |
| Oracle → RDS for Oracle | **DMS** |
| Oracle → different database engine | **SCT + DMS** |
| Oracle schema conversion | **SCT** |
| Oracle backup/recovery | **RMAN** |
| Oracle high availability | **RDS for Oracle Multi-AZ** |
| Oracle licenses already owned | **BYOL** |
| Oracle needs OS-level control | **EC2 / RDS Custom** |
| Aurora current writer | **Cluster / Writer endpoint** |
| Aurora read balancing | **Reader endpoint** |
| One specific Aurora instance | **Instance endpoint** |
| Different workloads → different Aurora instance groups | **Custom endpoints** |
| Custom Aurora endpoint limit | **5 per cluster** |
| Aurora replicas | **Up to 15 + 1 primary** |
| Aurora cluster storage | **Up to 256 TiB** |
| spiky / intermittent load | **Aurora Serverless v2** |
| cross-Region DR / low-latency global reads | **Aurora Global Database** |
| quick production copy for staging | **Aurora Cloning** |
| rewind Aurora MySQL after a mistake | **Aurora Backtrack** |

The key Aurora memory is:

```text
Aurora Cluster
│
├── Writer endpoint
│   → current writer
│
├── Reader endpoint
│   → balances read connections across readers
│
├── Instance endpoint
│   → one specific DB instance
│
└── Custom endpoint
    → specific group of DB instances
```

And the key RDS memory is:

```text
Multi-AZ
= HIGH AVAILABILITY

Read Replica
= READ SCALING

RDS Proxy
= CONNECTION MANAGEMENT

Aurora Custom Endpoint
= ROUTE DIFFERENT WORKLOADS TO DIFFERENT AURORA INSTANCES
```
