# Section 9: RDS & Aurora

## The idea

**Amazon Relational Database Service (Amazon RDS)** is a managed relational database service. AWS manages infrastructure, operating system (OS) maintenance, backups, patching, and database setup.

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

→ **Amazon Elastic Compute Cloud (Amazon EC2)** or, for supported Oracle/SQL Server scenarios, **RDS Custom**.

---

# RDS vs Aurora

|                      | **RDS**                                        | **Aurora**                               |
| -------------------- | ---------------------------------------------- | ---------------------------------------- |
| Type                 | Managed relational database service            | AWS relational database engine           |
| Engines              | MySQL, PostgreSQL, MariaDB, Oracle, SQL Server | MySQL-compatible / PostgreSQL-compatible |
| Storage architecture | Traditional DB storage                         | Distributed shared cluster storage       |
| Storage scaling      | Depends on engine/storage configuration        | Automatically grows                      |
| Maximum storage      | Depends on engine                              | Up to **256 TiB** for supported versions |
| Replicas             | Read Replicas                                  | Aurora Replicas                          |
| Fast DB failover     | Multi-AZ                                       | Promote Aurora Replica                   |
| Cross-Region DR      | Cross-Region Read Replica                      | Aurora Global Database                   |

### Simple decision rule

```text
Standard relational database
→ RDS

Very large / rapidly growing relational workload
→ Aurora

Online Transaction Processing (OLTP) + complex SQL
→ RDS or Aurora

Analytics / data warehouse
→ Amazon Redshift

NoSQL
→ Amazon DynamoDB
```

**OLTP** = **Online Transaction Processing**: transactional workloads such as application requests, orders, payments, and account updates.

**ACID** = **Atomicity, Consistency, Isolation, Durability**: the standard properties associated with reliable database transactions.

**Important:** OLTP, ACID, and complex SQL alone do **not** automatically mean Aurora. Both RDS and Aurora can support these workloads. Very large and growing storage requirements can be an important reason to choose Aurora.

---

# Multi-AZ vs Read Replicas

**Multi-AZ** = **Multiple Availability Zones**.

An **Availability Zone (AZ)** is an isolated location within an AWS Region.

|                    | **Multi-AZ**                            | **Read Replica**                                               |
| ------------------ | --------------------------------------- | -------------------------------------------------------------- |
| Main purpose       | High availability                       | Read scaling                                                   |
| Replication        | Synchronous for standard Multi-AZ       | Asynchronous                                                   |
| Read from standby? | **No** for traditional Multi-AZ standby | **Yes**                                                        |
| Automatic failover | **Yes**                                 | **No** as normal RR feature                                    |
| Location           | Another AZ / depends on deployment      | Same Region, another AZ, or another Region depending on engine |

**Read Replica (RR)** = a separate copy of a database that primarily handles read traffic.

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

**PITR** = **Point-in-Time Recovery**: restore a database to a specific time within the backup retention period.

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

# Encryption in transit — RDS SQL Server

**Encryption in transit** protects data while it moves between the application and the RDS database.

For **RDS for Microsoft SQL Server**, use **SSL/TLS** to encrypt the connection between the client application and RDS. AWS supports two main approaches: force SSL for all connections or configure individual clients/connections to use SSL.

## 1. Force all connections to use SSL

Set:

```text
rds.force_ssl = 1
```

Then reboot the DB instance because the parameter is **static**.

```text
EC2 / application
       ↓
     SSL/TLS
       ↓
RDS SQL Server

rds.force_ssl = 1
→ unencrypted connections are not allowed
```

> **All in-flight data between EC2 and RDS must be encrypted → `rds.force_ssl = 1`**

## 2. Configure the client application to use SSL

For client-specific encryption:

```text
RDS SQL Server certificate / CA
          ↓
Download certificate
          ↓
Import into client/server trust store
          ↓
Configure application / connection to use SSL
```

The application can then establish an encrypted SSL/TLS connection to RDS. AWS documents importing the appropriate RDS certificate into the client and enabling encrypted connections.

### Exam pattern

> **All EC2 → RDS SQL Server connections must use encryption in transit**

→ **`rds.force_ssl = 1`**

and, when the choices ask for client-side SSL configuration:

→ **Download/import the RDS CA certificate + configure the application to use SSL**

### Do not confuse these

| Requirement                                  | Solution                                                               |
| -------------------------------------------- | ---------------------------------------------------------------------- |
| Encrypt data **in transit**                  | **SSL/TLS**                                                            |
| Force all SQL Server connections to use SSL  | **`rds.force_ssl = 1`**                                                |
| Client needs to verify/trust RDS certificate | **Import RDS CA certificate into client trust store**                  |
| Encrypt data **at rest**                     | **RDS encryption / TDE where supported**                               |
| Control network access                       | **Security Groups**                                                    |
| Database authentication                      | **Password / IAM DB authentication / supported authentication method** |

**TDE** does not encrypt network traffic; it is an encryption-at-rest feature. Security Groups also do not encrypt traffic.

---

# Database Authentication

RDS supports several database authentication methods depending on the engine:

* **Password authentication** → traditional database username/password.
* **IAM Database Authentication** → temporary authentication token generated through **AWS Identity and Access Management (IAM)** instead of a database password.
* **Kerberos authentication** → external authentication through Kerberos / Microsoft Active Directory for supported engines.

## IAM Database Authentication

Use when the requirement says:

* Temporary / short-lived database authentication
* IAM user or role must connect directly to RDS
* Avoid storing database passwords in applications
* RDS MySQL / PostgreSQL / MariaDB

Flow:

```text
IAM User / Role
      ↓
IAM permissions
      ↓
Generate authentication token
      ↓
RDS database
      ↓
Temporary DB authentication
```

### Key facts

* The token is valid for **15 minutes**.
* The token is used **instead of a database password**.
* Authentication is managed through IAM, so the application does not need to store a long-lived DB password.
* The IAM policy needs permission for **`rds-db:connect`**.
* AWS Command Line Interface (**AWS CLI**) and AWS Software Development Kits (**AWS SDKs**) can generate/sign the token.
* IAM DB authentication can also be used from services such as **AWS Lambda**.

### MySQL / MariaDB

IAM authentication uses the AWS-provided:

**`AWSAuthenticationPlugin`**

Example concept:

```text
MySQL user
   ↓
AWSAuthenticationPlugin
   ↓
IAM authentication token
```

The database account is created with the AWS authentication plugin instead of a normal password.

### Important distinction

| Requirement                                 | Solution                                                                    |
| ------------------------------------------- | --------------------------------------------------------------------------- |
| Temporary token to connect to RDS           | **IAM Database Authentication**                                             |
| Store/rotate database passwords             | **AWS Secrets Manager**                                                     |
| Control whether an IAM identity can connect | **IAM policy / `rds-db:connect`**                                           |
| Network access to the DB                    | **Security Group**                                                          |
| Workforce SSO / AWS application access      | **IAM Identity Center**                                                     |
| MFA-based AWS authentication                | **Multi-Factor Authentication (MFA)**; not the RDS database-token mechanism |

**SSO** = **Single Sign-On**.

**MFA** = **Multi-Factor Authentication**.

**THE trap:** Secrets Manager does **not** generate IAM DB authentication tokens. It is used to store and rotate database credentials.

**THE trap:** MFA does **not** replace IAM Database Authentication for RDS.

**THE trap:** IAM Identity Center is not the direct mechanism used to authenticate a user to an RDS MySQL database.

> RDS MySQL + short-lived authentication token → **IAM Database Authentication**

> MySQL + `AWSAuthenticationPlugin` → **IAM Database Authentication**

> Database password must be stored and rotated → **Secrets Manager**

> IAM identity must be allowed to connect → **`rds-db:connect`**

---

# RDS Proxy

**Amazon RDS Proxy = managed database connection pool.**

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

| Tool            | Full name / purpose                                                           |
| --------------- | ----------------------------------------------------------------------------- |
| **AWS DMS**     | **AWS Database Migration Service** — migrate/replicate database data          |
| **AWS SCT**     | **AWS Schema Conversion Tool** — convert schema/code between database engines |
| **Oracle RMAN** | **Oracle Recovery Manager** — Oracle backup/recovery                          |

```text
Oracle → RDS Oracle
→ DMS

Oracle → PostgreSQL
→ SCT + DMS

Oracle backup/recovery
→ RMAN
```

### What each tool does

**AWS Database Migration Service (AWS DMS)**

→ Moves or continuously replicates database data.

**AWS Schema Conversion Tool (AWS SCT)**

→ Converts database schemas, code, and other database objects when moving between different database engines.

**Oracle Recovery Manager (RMAN)**

→ Oracle's backup and recovery tool.

---

# Oracle High Availability

**High Availability (HA)** means keeping the database available despite infrastructure failure.

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

**BYOL = Bring Your Own License**

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

**Amazon Aurora** is AWS's managed relational database engine compatible with:

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

## Aurora vs RDS Multi-AZ

| Requirement                   | RDS MySQL                     | Aurora                                                   |
| ----------------------------- | ----------------------------- | -------------------------------------------------------- |
| Automatic AZ failover         | **Multi-AZ**                  | **Aurora reader + automatic failover**                   |
| Storage replicated across AZs | Multi-AZ standby architecture | **Aurora storage automatically replicated across 3 AZs** |
| Read scaling                  | Read Replicas                 | **Aurora Replicas**                                      |
| Cross-Region DR               | Cross-Region Read Replica     | Aurora Global Database                                   |

---

# Aurora PostgreSQL — Babelfish

**Babelfish for Aurora PostgreSQL** is a compatibility feature that helps applications originally built for **Microsoft SQL Server** work with **Aurora PostgreSQL** with fewer application code changes.

It supports the **SQL Server Tabular Data Stream (TDS) protocol** and commonly used **Transact-SQL (T-SQL)** functionality.

### Main use case

```text
Existing SQL Server application
          ↓
     Aurora PostgreSQL
          +
       Babelfish
```

Instead of rewriting the application completely for PostgreSQL, Babelfish allows many existing SQL Server applications to continue using their SQL Server-compatible connection protocol and syntax.

### SAA exam signal

> **SQL Server → Aurora PostgreSQL + minimize application code changes**

→ **Babelfish**

### Typical migration architecture

```text
SQL Server
   │
   ├── AWS SCT
   │      ↓
   │   Convert schema
   │
   └── AWS DMS
          ↓
Aurora PostgreSQL
      + Babelfish
          ↓
Existing application
```

The tools have different jobs:

| Requirement                                     | Service       |
| ----------------------------------------------- | ------------- |
| SQL Server compatibility with Aurora PostgreSQL | **Babelfish** |
| Convert schema/database objects                 | **AWS SCT**   |
| Migrate/replicate database data                 | **AWS DMS**   |

### Important distinction

**Babelfish is not a database migration service.**

It provides **compatibility** so the existing application can communicate with Aurora PostgreSQL with fewer code modifications.

```text
Babelfish
→ application compatibility

SCT
→ schema conversion

DMS
→ data migration
```

### Example

> A company uses SQL Server and wants to migrate to Aurora PostgreSQL while minimizing application code modifications.

→ **Babelfish + AWS SCT + AWS DMS**

The exact combination depends on the question's answer choices. If the question asks for the two actions that achieve the migration:

> **Enable Babelfish on Aurora PostgreSQL**

*

> **Use AWS SCT for schema conversion and AWS DMS for data migration**

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

**Aurora flips the canonical name record (CNAME) for your DB Instance to point at the healthy replica, which in turn is promoted to become the new primary.**

**CNAME** = **Canonical Name record**, a DNS record that points one domain name to another canonical name.

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

**Aurora Serverless v2** automatically adjusts the **compute capacity** of Aurora Serverless writer and reader instances based on workload.

Good for:

* Unpredictable workloads
* Spiky workloads
* Intermittent workloads
* Workloads that do not need fixed capacity

```text
Low workload
→ lower ACU capacity

Traffic spike
→ higher ACU capacity

Traffic drops
→ lower ACU capacity
```

**ACU** = **Aurora Capacity Unit**.

Aurora Serverless v2 scales the capacity of an existing serverless writer or reader within the configured minimum/maximum ACU range. It is designed for variable and unpredictable workloads such as e-commerce sales events.

> Unpredictable/intermittent Aurora workload → **Aurora Serverless v2**

---

## Aurora Serverless v2 vs Aurora Read Replica Auto Scaling

These are easy to confuse.

### Aurora Serverless v2

Scales the **compute capacity of a writer or reader**:

```text
One Aurora Serverless writer

2 ACUs
  ↓
4 ACUs
  ↓
8 ACUs
  ↓
16 ACUs
```

Think:

> **"The database instance itself needs more CPU/memory capacity."**

### Aurora Read Replica Auto Scaling

Adds or removes **reader instances**:

```text
Aurora cluster

Writer
  │
  ├── Reader 1
  ├── Reader 2
  └── Reader 3

Read traffic increases
        ↓
Auto Scaling
        ↓
Add Reader 4
```

Think:

> **"There are too many read requests; add more readers."**

Aurora can automatically add and remove Aurora Replicas based on configured performance metrics.

### The simplest distinction

```text
Aurora Serverless v2
→ Scale UP/DOWN compute capacity
→ ACUs
→ unpredictable overall workload

Aurora Replica Auto Scaling
→ Scale OUT/IN reader instances
→ number of readers
→ read-heavy workload
```

### Important

They are **not mutually exclusive**.

An Aurora Serverless cluster can also have readers, and those readers can provide horizontal read scaling.

### Exam shortcut

> **Unpredictable database compute demand → Aurora Serverless v2**

> **Read-heavy workload → Aurora Read Replicas + Auto Scaling**

> **Writer itself needs more capacity → Aurora Serverless v2**

> **Need more reader capacity → Read Replicas / Reader Auto Scaling**

---

# Aurora Global Database

Designed for **cross-Region** architectures.

Use for:

* Cross-Region Disaster Recovery (DR)
* Very low replication lag
* Fast recovery after Regional failure
* Low-latency reads across Regions

**DR = Disaster Recovery**: recovering an application/database after a major failure such as a Regional outage.

```text
Primary Region
      ↓
Aurora Global Database
      ↓
Secondary Region
```

Secondary Regions can also serve reads.

> Aurora + cross-Region DR + very low RPO / fast recovery → **Aurora Global Database**

**RPO = Recovery Point Objective**: how much recent data loss is acceptable after a failure.

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

| Service                    | Type        | Clue                               |
| -------------------------- | ----------- | ---------------------------------- |
| **RDS**                    | Relational  | SQL                                |
| **Aurora**                 | Relational  | MySQL/PostgreSQL-compatible        |
| **DynamoDB Global Tables** | NoSQL       | Multi-Region NoSQL                 |
| **Amazon Timestream**      | Time-series | Internet of Things (IoT) / metrics |

**NoSQL** = **non-relational database**.

**IoT** = **Internet of Things**, such as connected devices and sensors.

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

> A database must remain available if an Availability Zone fails, and the application should automatically connect to the database after the failure.
> → **Multi-AZ**

> An application has a large number of read requests and the primary database is becoming overloaded. The company wants to offload read traffic to separate database instances.
> → **Read Replica**

> A serverless application using Lambda creates a very large number of short-lived connections to an RDS database, causing connection-limit problems.
> → **RDS Proxy**

> An RDS database is growing unpredictably and the company wants storage capacity to increase automatically when the database approaches its storage limit.
> → **RDS Storage Auto Scaling**

> A company accidentally deleted or changed data at 14:30 and needs to restore the database to its state at 14:25.
> → **Automated backups / Point-in-Time Recovery (PITR)**

> A company must retain a database backup for several years and does not want AWS to delete it automatically.
> → **Manual snapshot**

> An existing RDS database is unencrypted and the company now requires encryption at rest.
> → **Snapshot → encrypted snapshot copy → restore encrypted DB**

> An application must connect to RDS using a temporary authentication token instead of storing a permanent database password.
> → **IAM Database Authentication**

> An RDS MySQL database uses `AWSAuthenticationPlugin` for database login.
> → **IAM Database Authentication**

> An IAM role must be allowed to authenticate directly to an RDS database using IAM database authentication.
> → **`rds-db:connect`**

> A company wants to securely store database credentials and automatically rotate the database password.
> → **AWS Secrets Manager**

> A company wants to encrypt all in-flight connections between EC2 web servers and an RDS for SQL Server database.
> → **Enable SSL/TLS; for all connections set `rds.force_ssl = 1`**

> A company wants all RDS for SQL Server connections to use SSL/TLS and the application/client must trust the RDS certificate.
> → **Set `rds.force_ssl = 1` + configure the client/application with the RDS CA certificate**

> A company wants encryption in transit, not encryption at rest.
> → **SSL/TLS**

> A company configures TDE but the requirement is to encrypt traffic between EC2 and RDS.
> → **Wrong**; TDE protects data at rest.

> A company configures a Security Group to allow the database port and assumes the traffic is encrypted.
> → **Wrong**; Security Groups control network access but do not encrypt traffic.

> A company wants to migrate an Oracle database to Amazon RDS for Oracle while keeping Oracle as the database engine.
> → **AWS DMS**

> A company wants to migrate an Oracle database to PostgreSQL and needs to convert the schema and database code before moving the data.
> → **AWS SCT + AWS DMS**

> A database administrator needs an Oracle-specific tool for database backup and recovery.
> → **Oracle RMAN**

> An Oracle database must automatically fail over to a standby database when its Availability Zone fails.
> → **RDS for Oracle Multi-AZ**

> A company already owns eligible Oracle licenses and wants to use those licenses with Amazon RDS for Oracle.
> → **BYOL**

> A database workload requires OS-level access and custom server configuration that standard managed RDS does not allow.
> → **EC2 / RDS Custom**

> A company wants to migrate a SQL Server application to Aurora PostgreSQL while minimizing application code modifications.
> → **Babelfish**

> A company needs to convert SQL Server database schemas and database objects before moving them to PostgreSQL.
> → **AWS SCT**

> A company needs to migrate the actual database data from one database engine to another.
> → **AWS DMS**

> An application must always connect to whichever Aurora DB instance is currently the primary writer, including after a failover.
> → **Cluster / Writer endpoint**

> An Aurora cluster has multiple Replicas and the application needs to distribute read connections across them.
> → **Reader endpoint**

> An administrator needs to connect directly to one specific Aurora DB instance rather than the cluster as a whole.
> → **Instance endpoint**

> Production, reporting, and analytics workloads need to connect to different selected groups of Aurora DB instances.
> → **Custom endpoint**

> The primary Aurora DB instance fails and the cluster already has an Aurora Replica available.
> → **Promote the Aurora Replica**

> An Aurora cluster has only one DB instance and that instance fails. There is no Aurora Replica available for promotion.
> → **Recreate the primary DB instance**

> An Aurora database has unpredictable traffic with long periods of low usage and occasional spikes, and the company does not want to manage fixed database capacity.
> → **Aurora Serverless v2**

> A company has an unpredictable workload and the **database writer itself** may suddenly need more CPU and memory capacity.
> → **Aurora Serverless v2**

> A company has a **read-heavy workload** and wants the number of Aurora reader instances to increase or decrease automatically based on load.
> → **Aurora Read Replicas + Auto Scaling**

> A company says "Aurora Auto Scaling" but the requirement is to automatically give the existing writer more CPU/memory capacity rather than add readers.
> → **Aurora Serverless v2**

> A company wants to scale the **number of read replicas**, not the compute capacity of one existing DB instance.
> → **Aurora Read Replica Auto Scaling**

> A company has an Aurora Serverless workload and also needs more read capacity.
> → **Aurora Serverless v2 + Aurora readers can be used together**

> A company runs Aurora in one Region and needs cross-Region disaster recovery, very low replication lag, and read access from another Region.
> → **Aurora Global Database**

> A standard RDS database must have a copy in another AWS Region for disaster recovery and possible read scaling.
> → **Cross-Region Read Replica**

> Developers need a fast production-like copy of an Aurora database for testing and development.
> → **Aurora Cloning**

> A team accidentally made changes to an Aurora MySQL database and wants to rewind it to an earlier point without performing a traditional restore.
> → **Aurora Backtrack**

> A company needs a relational database for application transactions, orders, payments, and complex SQL queries, but there is no special requirement for Aurora features.
> → **RDS or Aurora**

> A relational database workload requires very large and rapidly growing storage capacity.
> → **Aurora**

> A company needs a managed data warehouse for large-scale analytical queries.
> → **Amazon Redshift**

> An application requires a NoSQL database.
> → **Amazon DynamoDB**

> A company needs a multi-Region NoSQL database with data available across multiple AWS Regions.
> → **DynamoDB Global Tables**

> An application stores time-series data such as IoT sensor measurements and application metrics.
> → **Amazon Timestream**

> An **Aurora** cluster needs highly available storage across Availability Zones.
> → **Aurora automatically replicates storage across 3 Availability Zones**

---

# Pocket card

| Keyword                                   | Answer                                                                      |
| ----------------------------------------- | --------------------------------------------------------------------------- |
| AZ failure                                | **Multi-AZ**                                                                |
| Automatic regional failover               | **Multi-AZ**                                                                |
| Scale reads                               | **Read Replica**                                                            |
| Cross-Region RDS DR                       | **Cross-Region Read Replica**                                               |
| Aurora cross-Region DR                    | **Aurora Global Database**                                                  |
| Very low RPO + fast cross-Region recovery | **Aurora Global Database**                                                  |
| Lambda + too many DB connections          | **RDS Proxy**                                                               |
| Point-in-time restore                     | **Automated backup / PITR**                                                 |
| Long-term backup                          | **Manual snapshot**                                                         |
| Existing unencrypted RDS → encrypted      | **Snapshot → encrypted copy → restore**                                     |
| Encrypt RDS SQL Server in transit         | **SSL/TLS**                                                                 |
| Force SQL Server connections to SSL       | **`rds.force_ssl = 1`**                                                     |
| Client trusts RDS certificate             | **Import RDS CA certificate + enable SSL/TLS**                              |
| Temporary DB auth token                   | **IAM Database Authentication**                                             |
| IAM DB token lifetime                     | **15 minutes**                                                              |
| MySQL IAM authentication                  | **`AWSAuthenticationPlugin`**                                               |
| Allow IAM identity to connect             | **`rds-db:connect`**                                                        |
| Store/rotate DB passwords                 | **Secrets Manager**                                                         |
| Oracle → RDS Oracle                       | **AWS Database Migration Service (DMS)**                                    |
| Oracle → different engine                 | **AWS Schema Conversion Tool (SCT) + AWS Database Migration Service (DMS)** |
| Oracle backup/recovery                    | **Oracle Recovery Manager (RMAN)**                                          |
| Oracle AZ HA                              | **RDS Multi-AZ**                                                            |
| Existing Oracle license                   | **Bring Your Own License (BYOL)**                                           |
| OS-level DB control                       | **EC2 / RDS Custom**                                                        |
| SQL Server → Aurora PostgreSQL            | **Babelfish**                                                               |
| SQL Server schema conversion              | **AWS Schema Conversion Tool (SCT)**                                        |
| Database data migration                   | **AWS Database Migration Service (DMS)**                                    |
| Aurora current writer                     | **Writer/Cluster endpoint**                                                 |
| Aurora read balancing                     | **Reader endpoint**                                                         |
| One Aurora instance                       | **Instance endpoint**                                                       |
| Different Aurora instance groups          | **Custom endpoint**                                                         |
| Aurora failure + Replica exists           | **Promote Replica**                                                         |
| Aurora failure + no Replica               | **Recreate primary instance**                                               |
| Spiky/unpredictable Aurora workload       | **Aurora Serverless v2**                                                    |
| Writer needs more CPU/memory              | **Aurora Serverless v2**                                                    |
| Scale Aurora compute with ACUs            | **Aurora Serverless v2**                                                    |
| Read-heavy workload                       | **Aurora Read Replicas**                                                    |
| Automatically add/remove readers          | **Aurora Replica Auto Scaling**                                             |
| Scale UP/DOWN existing DB compute         | **Aurora Serverless v2**                                                    |
| Scale OUT/IN number of readers            | **Aurora Read Replica Auto Scaling**                                        |
| Serverless + additional read capacity     | **Serverless v2 + Aurora readers**                                          |
| Aurora cross-Region DR                    | **Aurora Global Database**                                                  |
| Quick Aurora copy                         | **Aurora Cloning**                                                          |
| Rewind Aurora MySQL                       | **Aurora Backtrack**                                                        |
| Multi-Region NoSQL                        | **DynamoDB Global Tables**                                                  |
| Time-series database                      | **Amazon Timestream**                                                       |
| Aurora distributed storage                | **Replicated across 3 AZs**                                                 |
