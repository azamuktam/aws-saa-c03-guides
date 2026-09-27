# Section 9: RDS & Aurora

## The idea

**Amazon RDS** is a managed relational database service. AWS manages infrastructure, OS maintenance, backups, patching, and database setup.

Supported engines:

* MySQL
* PostgreSQL
* MariaDB
* Oracle
* SQL Server

RDS does **not** provide normal OS-level access.

Need:

* Full OS/server control
* Custom database software
* OS-level configuration

→ **EC2** or, for supported Oracle/SQL Server scenarios, **RDS Custom**.

---

# Multi-AZ vs Read Replicas

|                    | **Multi-AZ**                       | **Read Replica**                                               |
| ------------------ | ---------------------------------- | -------------------------------------------------------------- |
| Main purpose       | High availability                  | Read scaling                                                   |
| Replication        | Synchronous for standard Multi-AZ  | Asynchronous                                                   |
| Read from standby? | **No**                             | **Yes**                                                        |
| Automatic failover | **Yes**                            | **No** as normal RR feature                                    |
| Location           | Another AZ / depends on deployment | Same Region, another AZ, or another Region depending on engine |

### Multi-AZ

```text
Primary
  │ synchronous
  ▼
Standby → automatic failover
```

Standby is **not for normal reads**.

### Read Replica

```text
Primary
 ├──→ Replica
 ├──→ Replica
 └──→ Replica
```

Used for read traffic.

### Exam patterns

> AZ failure + automatic failover → **Multi-AZ**

> Read-heavy workload → **Read Replica**

> Use Multi-AZ standby for reads → **Wrong**

> Read replica automatically replaces primary → **Wrong**; promotion is separate.

You can use both:

```text
Multi-AZ → availability
Read Replica → read scaling
```

---

# Backups and recovery

## Automated backups

Provide **Point-in-Time Recovery (PITR)**.

Standard RDS DB instance retention:

**0–35 days**

> Restore to a specific point in time → **Automated backups / PITR**

## Manual snapshots

Remain until deleted.

Use for:

* Long-term retention
* Compliance
* Months/years of retention

> Keep backup for 10 years → **Manual snapshot**

---

# Encrypting RDS

Enable encryption when creating the database.

Existing unencrypted DB:

```text
Unencrypted DB
 ↓
Snapshot
 ↓
Copy snapshot with encryption
 ↓
Restore encrypted DB
```

You cannot simply enable encryption on an existing unencrypted RDS DB instance.

---

# RDS Proxy

**RDS Proxy = managed database connection pool.**

Useful when many short-lived applications, especially Lambda, create too many connections.

```text
Lambda → RDS Proxy → RDS
```

* Pools connections
* Reduces connection overhead
* Helps prevent connection storms

> Lambda creates too many RDS connections → **RDS Proxy**

```text
RDS Proxy → connection management
Read Replica → read scaling
```

---

# RDS Storage Auto Scaling

Automatically increases allocated storage as the database approaches its threshold.

> Storage is growing unpredictably → **RDS Storage Auto Scaling**

---

# Oracle on RDS

RDS supports Oracle.

```text
On-prem Oracle
    ↓
   DMS
    ↓
RDS for Oracle
```

## Oracle migration: DMS vs SCT vs RMAN

| Tool            | Purpose                                      |
| --------------- | -------------------------------------------- |
| **AWS DMS**     | Migrate/replicate database data              |
| **AWS SCT**     | Convert schema/code between database engines |
| **Oracle RMAN** | Oracle backup/recovery                       |

```text
Oracle → RDS Oracle
→ DMS

Oracle → PostgreSQL
→ SCT + DMS

Oracle backup/recovery
→ RMAN
```

---

# Oracle High Availability

Oracle database + AZ failure + automatic failover:

→ **RDS for Oracle Multi-AZ**

```text
AZ-A
Primary
  │ synchronous
  ▼
AZ-B
Standby
```

---

# Oracle licensing

### License Included

AWS provides the Oracle license under the supported RDS model.

### BYOL

**Bring Your Own License**

> Company already owns eligible Oracle licenses → **BYOL**

---

# Oracle and OS control

Need:

* OS-level access
* Custom Oracle configuration requiring server access
* Full underlying server control

→ **EC2** or **RDS Custom for Oracle**, depending on the scenario.

---

# Aurora

**Aurora** is AWS's managed relational database engine compatible with:

* MySQL
* PostgreSQL

Its key architectural difference from standard RDS is **shared cluster storage**.

```text
Aurora Cluster
├── Primary / Writer
├── Aurora Replica
├── Aurora Replica
└── Aurora Replica
        ↓
  Shared cluster storage
```

Aurora storage is automatically replicated across multiple AZs.

Aurora supports:

**1 primary + up to 15 Aurora Replicas**

---

# Aurora endpoints

| Endpoint                      | Purpose                                     |
| ----------------------------- | ------------------------------------------- |
| **Cluster / Writer endpoint** | Current primary/writer                      |
| **Reader endpoint**           | Distribute read connections across replicas |
| **Instance endpoint**         | One specific DB instance                    |
| **Custom endpoint**           | Selected group of Aurora instances          |

## Cluster / Writer endpoint

> Always connect to the current Aurora writer → **Cluster / Writer endpoint**

The endpoint continues to provide access to the current writer after failover.

## Reader endpoint

Distributes **read connections** across Aurora Replicas.

> Balance read traffic across replicas → **Reader endpoint**

**Important:** balances **connections**, not individual SQL queries.

## Instance endpoint

> Connect to one specific Aurora instance → **Instance endpoint**

## Custom endpoint

Routes connections to a selected group of Aurora instances.

Example:

```text
Reader A + B → Production
Reader C + D → Reporting
```

> Different workloads should use different Aurora instance groups → **Custom endpoint**

---

# Aurora Failover

Aurora failover behavior depends on whether the cluster has an **Aurora Replica**.

## Aurora with an Aurora Replica

If the primary DB instance fails:

```text
Primary fails
     ↓
Aurora Replica
     ↓
Replica is promoted
     ↓
New primary
     ↓
Cluster / Writer endpoint points to new primary
```

Aurora promotes an existing Aurora Replica to become the new primary. Failover is much faster than creating a new DB instance.

> **Primary failure + Aurora Replica** → **Promote the Aurora Replica**

For high availability, Aurora Replicas should ideally be placed in different Availability Zones.

---

## Aurora with only one DB instance

If there is **no Aurora Replica**:

```text
Primary fails
     ↓
No Replica to promote
     ↓
Aurora recreates the primary
     ↓
Same Availability Zone
     ↓
Service becomes available again
```

Aurora automatically attempts to create a new primary DB instance in the **same Availability Zone** when there are no Aurora Replicas.

> **Single Aurora instance + failure** → **Create replacement DB instance in the same AZ**

This recovery is significantly slower than promoting an existing Replica.

### Very important distinction

```text
Aurora storage
→ distributed across multiple AZs

Aurora DB instance
→ database compute

Replica exists
→ fast failover by promotion

No Replica
→ recreate the DB instance
```

The distributed Aurora storage **does not mean another DB instance automatically exists in every AZ**.

### Exam pattern

> **Aurora primary fails + Replica exists**

→ **Promote Replica**

> **Aurora primary fails + only one DB instance**

→ **Create replacement instance in the same AZ**

> **Aurora has distributed storage**

→ **Does NOT mean a DB instance automatically exists in each AZ**

---

# Aurora Serverless v2

Automatically adjusts database capacity with workload.

Good for:

* Unpredictable workloads
* Spiky workloads
* Intermittent workloads
* Workloads that do not need fixed capacity

> Unpredictable/intermittent Aurora workload → **Aurora Serverless v2**

---

# Aurora Global Database

Designed for **cross-Region** architectures.

Use for:

* Cross-Region DR
* Very low replication lag
* Fast recovery after Regional failure
* Low-latency reads across Regions

```text
Primary Region
      ↓
Aurora Global Database
      ↓
Secondary Region
```

Secondary Regions can also serve reads.

> Aurora + cross-Region DR + very low RPO / fast recovery → **Aurora Global Database**

---

# Aurora Cloning

**Aurora Cloning = fast copy using copy-on-write.**

Use for:

* Testing
* Development
* Staging

> Quick production-like Aurora copy → **Aurora Cloning**

---

# RDS cross-Region Read Replicas

Standard RDS engines can use **cross-Region Read Replicas**.

```text
Primary Region
      ↓ asynchronous replication
Secondary Region
RDS Read Replica
```

Use for:

* Cross-Region DR
* Read scaling
* Maintaining a copy in another Region

Replication is **asynchronous**, so lag is possible.

```text
Multi-AZ
→ regional high availability

Cross-Region Read Replica
→ cross-Region DR / read scaling
```

---

# Cross-Region DR: RDS Read Replica vs Aurora Global Database

| Solution                          | Database                | Main use                                      |
| --------------------------------- | ----------------------- | --------------------------------------------- |
| **RDS cross-Region Read Replica** | RDS relational engines  | Cross-Region DR / read scaling                |
| **Aurora Global Database**        | Aurora MySQL/PostgreSQL | Cross-Region DR + very low lag + global reads |

### Decision rule

> Cross-Region DR for RDS → **Cross-Region Read Replica**

> Aurora + very low RPO + very fast recovery → **Aurora Global Database**

---

# Relational vs non-relational global databases

| Service                    | Type        | Clue                        |
| -------------------------- | ----------- | --------------------------- |
| **RDS**                    | Relational  | SQL                         |
| **Aurora**                 | Relational  | MySQL/PostgreSQL-compatible |
| **DynamoDB Global Tables** | NoSQL       | Multi-Region NoSQL          |
| **Timestream**             | Time-series | IoT / metrics               |

> Relational → **RDS / Aurora**

> Multi-Region NoSQL → **DynamoDB Global Tables**

> Time-series → **Timestream**

---

# Aurora Backtrack

**Aurora Backtrack** lets you rewind an **Aurora MySQL** database to an earlier point without a traditional restore to a new DB.

Example:

```text
10:00 → good
10:15 → accidental change
10:20 → discover mistake
       ↓
   Backtrack
       ↓
   earlier state
```

> Quickly undo accidental Aurora MySQL changes → **Aurora Backtrack**

---

# Aurora storage

Aurora uses a distributed **cluster volume** rather than traditional database-local storage.

It is:

* Replicated across multiple AZs
* Self-healing
* Shared by Aurora DB instances

Readers use the shared Aurora storage rather than maintaining independent full database copies.

```text
Aurora DB instances
        ↓
Shared cluster volume
        ↓
Copies across multiple AZs
```

The storage layer and database-instance layer are separate:

```text
Storage
→ distributed and highly durable

DB instances
→ compute
```

This is why Aurora can have highly durable storage even when only one DB instance exists. However, a Replica is still needed for **fast instance failover**.

---

# Question patterns

> **AZ failure + automatic failover** → **RDS Multi-AZ**

> **Reporting queries overload primary** → **Read Replica**

> **Lambda creates too many DB connections** → **RDS Proxy**

> **Storage grows unpredictably** → **RDS Storage Auto Scaling**

> **Restore to a specific point in time** → **Automated backups / PITR**

> **Keep backup for years** → **Manual snapshot**

> **Encrypt existing unencrypted RDS** → **Snapshot → encrypted copy → restore**

> **Oracle → RDS Oracle** → **DMS**

> **Oracle → different DB engine** → **SCT + DMS**

> **Oracle backup/recovery** → **RMAN**

> **Oracle AZ HA** → **RDS for Oracle Multi-AZ**

> **Existing Oracle license** → **BYOL**

> **OS-level Oracle control** → **EC2 / RDS Custom**

> **Aurora read balancing** → **Reader endpoint**

> **Current Aurora writer** → **Cluster / Writer endpoint**

> **One Aurora instance** → **Instance endpoint**

> **Different Aurora instance groups for workloads** → **Custom endpoint**

> **Aurora primary failure + Replica exists** → **Promote Aurora Replica**

> **Aurora primary failure + no Replica** → **Recreate primary DB instance**

> **Unpredictable Aurora workload** → **Aurora Serverless v2**

> **Aurora cross-Region DR** → **Aurora Global Database**

> **RDS cross-Region DR** → **Cross-Region Read Replica**

> **Quick Aurora copy** → **Aurora Cloning**

> **Rewind Aurora MySQL** → **Aurora Backtrack**

> **Multi-Region NoSQL** → **DynamoDB Global Tables**

> **Time-series database** → **Timestream**

---

# Pocket card

| Keyword                                   | Answer                                  |
| ----------------------------------------- | --------------------------------------- |
| AZ failure                                | **Multi-AZ**                            |
| Automatic regional failover               | **Multi-AZ**                            |
| Scale reads                               | **Read Replica**                        |
| Cross-Region RDS DR                       | **Cross-Region Read Replica**           |
| Aurora cross-Region DR                    | **Aurora Global Database**              |
| Very low RPO + fast cross-Region recovery | **Aurora Global Database**              |
| Lambda + too many DB connections          | **RDS Proxy**                           |
| Point-in-time restore                     | **Automated backup / PITR**             |
| Long-term backup                          | **Manual snapshot**                     |
| Existing unencrypted RDS → encrypted      | **Snapshot → encrypted copy → restore** |
| Oracle → RDS Oracle                       | **DMS**                                 |
| Oracle → different engine                 | **SCT + DMS**                           |
| Oracle backup/recovery                    | **RMAN**                                |
| Oracle AZ HA                              | **RDS Multi-AZ**                        |
| Existing Oracle license                   | **BYOL**                                |
| OS-level DB control                       | **EC2 / RDS Custom**                    |
| Aurora current writer                     | **Writer/Cluster endpoint**             |
| Aurora read balancing                     | **Reader endpoint**                     |
| One Aurora instance                       | **Instance endpoint**                   |
| Different Aurora instance groups          | **Custom endpoint**                     |
| Aurora failure + Replica exists           | **Promote Replica**                     |
| Aurora failure + no Replica               | **Recreate primary instance**           |
| Spiky/unpredictable Aurora workload       | **Serverless v2**                       |
| Aurora cross-Region DR                    | **Global Database**                     |
| Quick Aurora copy                         | **Cloning**                             |
| Rewind Aurora MySQL                       | **Backtrack**                           |
| Multi-Region NoSQL                        | **DynamoDB Global Tables**              |
| Time-series database                      | **Timestream**                          |

