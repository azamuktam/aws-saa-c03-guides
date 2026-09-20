# Section 35: Disaster Recovery

## The big idea

**Disaster Recovery (DR)** is about one thing:

> **If the primary system fails completely, how quickly can we recover, and how much data can we afford to lose?**

Two numbers define the requirement:

### RPO — Recovery Point Objective

**RPO = maximum acceptable data loss**

It answers:

> **"How much data can we lose?"**

Example:

* Database is backed up every night.
* A disaster happens at 5 PM.
* The latest backup is from midnight.
* Up to **17 hours of data** could be lost.

So the RPO is **up to 17 hours in this specific failure**; the backup schedule gives a maximum RPO of about **24 hours**.

```text
last backup                         disaster
    ●---------------------------------✖
    ◀──────────── possible data loss ────────────▶
                         = RPO
```

### RTO — Recovery Time Objective

**RTO = maximum acceptable downtime**

It answers:

> **"How long can the application stay offline?"**

Example:

* Disaster happens at 5 PM.
* Business requires the application back by 5:30 PM.
* Maximum allowed downtime = **30 minutes**.

So:

**RPO = data loss**
**RTO = downtime**

A useful memory trick:

> **RPO = Point = data**
> **RTO = Time = downtime**

---

# The four main DR strategies

The easiest way to understand them is to imagine that your production environment is a car.

* **Backup & Restore** → no spare car
* **Pilot Light** → only the engine is ready
* **Warm Standby** → a smaller spare car is already running
* **Multi-Site Active-Active** → two full cars are running at the same time

The faster you need recovery to be, the more you normally have to keep running in the DR environment.

| Strategy                     | Typical RTO             | Typical RPO          | Cost | What exists in the DR region?             |
| ---------------------------- | ----------------------- | -------------------- | ---- | ----------------------------------------- |
| **Backup & Restore**         | Hours                   | Hours                | $    | Backups only                              |
| **Pilot Light**              | Tens of minutes or more | Minutes–seconds      | $$   | Critical components, usually data layer   |
| **Warm Standby**             | Minutes                 | Seconds–minutes      | $$$  | Smaller but fully functional environment  |
| **Multi-Site Active-Active** | Very low / near-zero    | Very low / near-zero | $$$$ | Full environment actively serving traffic |

> **Important:** These are general patterns, not hard AWS guarantees. Actual RTO/RPO depends on the architecture and how quickly you can detect, promote, scale, and redirect traffic.

---

# 1. Backup & Restore

### The idea

Nothing significant is running in the DR region.

You simply keep backups somewhere safe, often in Amazon S3 or a backup vault.

When disaster happens:

1. Restore the database.
2. Recreate infrastructure.
3. Deploy the application.
4. Restore data.
5. Redirect traffic.

```text
PRIMARY REGION
   Application
   Database
       |
       | backups
       ↓
   S3 / Backup

DR REGION
   Nothing running
       |
       | disaster
       ↓
   Rebuild everything
```

### When to use it

Use it when:

* **Cost must be as low as possible**
* **Hours of downtime are acceptable**
* Some data loss is acceptable

### Exam signal

> "Lowest cost, several hours of downtime is acceptable"

→ **Backup & Restore**

### Memory

> **Nothing is ready. Rebuild after the disaster.**

---

# 2. Pilot Light

### The idea

The DR environment keeps only the **most important core components** running.

Typically:

* Database is continuously replicated
* Critical data is already available
* Application servers are **not running at full capacity**
* Compute resources are started or scaled up after disaster

Think:

> **The pilot light is on, but the whole house is not running.**

```text
PRIMARY REGION
   Application
       ↓
   Primary DB
       │
       │ replication
       ↓
DR REGION
   Standby DB
   Minimal infrastructure
   EC2 / application = OFF or minimal
```

When disaster happens:

1. Promote/start the DR database.
2. Launch or scale application servers.
3. Recreate remaining resources.
4. Redirect users.

### When to use it

Good when:

* You need **faster recovery than backup/restore**
* You want to keep costs below warm standby
* The database or core data must already be synchronized

### Exam signal

> "Database is continuously replicated, but compute is launched only after the disaster."

→ **Pilot Light**

### Memory

> **Core is alive; compute is mostly off.**

---

# 3. Warm Standby

### The idea

The **entire application stack already exists in the DR region** and is working, but at reduced capacity.

For example:

```text
PRIMARY REGION
10 EC2 instances
      ↓
   Database

DR REGION
2 EC2 instances
      ↓
Standby database
```

The DR environment can already serve traffic, but it is intentionally **smaller**.

When disaster happens:

1. Redirect traffic to DR.
2. Scale the environment up.
3. Continue serving users.

### When to use it

Good when:

* RTO must be **minutes**
* You need a functioning application environment already running
* Some downtime is acceptable
* Active-active would be unnecessarily expensive

### Exam signal

> "A smaller version of the complete environment is always running."

→ **Warm Standby**

### Memory

> **Everything is ready, just smaller.**

---

# 4. Multi-Site Active-Active

### The idea

Both regions are **fully operational at the same time**.

Users are already being served from both regions.

```text
             ┌── Region A ──→ users
Users ───────┤
             └── Region B ──→ users
```

If Region A fails:

```text
Region A ✖

Users ─────────────→ Region B
```

There is no need to build or start the application from scratch.

### When to use it

Use it when:

* Downtime must be extremely low
* The business requires near-continuous availability
* Very little data loss is acceptable
* Cost is less important than availability

### Exam signal

> "Both regions are active and serving traffic."

→ **Multi-Site Active-Active**

### Memory

> **Both environments are live.**

---

# The most important comparison

Think of the four strategies as a ladder:

```text
                 FASTER RECOVERY
                       ↑
                       │
        Multi-Site Active-Active
                │
                │
             Warm Standby
                │
                │
             Pilot Light
                │
                │
          Backup & Restore
                       │
                       ↓
                 LOWER COST
```

Or remember:

```text
Backup & Restore
    ↓
Build after disaster

Pilot Light
    ↓
Data/core already ready

Warm Standby
    ↓
Whole environment ready, but smaller

Active-Active
    ↓
Everything already running
```

---

# How to choose the strategy on the exam

Use this process:

### Step 1 — Find the RPO

Ask:

> **How much data can we lose?**

Examples:

* Hours of data loss → Backup may be enough
* Minutes/seconds → replication is usually needed
* Near-zero → active replication / multi-site architecture

### Step 2 — Find the RTO

Ask:

> **How quickly must the application recover?**

Examples:

* Hours → Backup & Restore
* Tens of minutes → Pilot Light
* Minutes → Warm Standby
* Very low / near-zero → Active-Active

### Step 3 — Eliminate strategies that cannot meet the requirement

### Step 4 — Choose the cheapest remaining option

This is a very common exam pattern.

> **Do not automatically choose the most advanced architecture.**
>
> Choose the **cheapest strategy that satisfies the requirements**.

---

# Common exam patterns

### "RTO of several hours, lowest possible cost"

→ **Backup & Restore**

There is no reason to pay for a continuously running DR environment.

---

### "Database is continuously replicated, application servers are started only during a disaster"

→ **Pilot Light**

The critical data layer is already ready, but compute is not fully running.

---

### "A smaller version of the entire environment is always running"

→ **Warm Standby**

The complete stack exists and works, but at reduced capacity.

---

### "Both regions are serving production traffic"

→ **Multi-Site Active-Active**

Both sites are already active.

---

### "Application must recover within minutes and the DR environment must already be operational"

→ **Warm Standby**

Pilot Light may require additional startup/recovery steps.

---

### "Application must have extremely low downtime and both regions must remain available"

→ **Multi-Site Active-Active**

---

# AWS services commonly used for DR

The DR **strategy** tells you the architecture.

These AWS services are the **building blocks** used to implement it.

| Service                               | What it does                                                               |
| ------------------------------------- | -------------------------------------------------------------------------- |
| **Aurora Global Database**            | Replicates an Aurora database across regions with very low replication lag |
| **DynamoDB Global Tables**            | Multi-Region, multi-active DynamoDB replication                            |
| **S3 Cross-Region Replication (CRR)** | Automatically replicates S3 objects to another region                      |
| **AWS Backup**                        | Centrally manages backups across supported AWS services                    |
| **AWS Backup Vault Lock**             | Helps make backups immutable / protected from deletion                     |
| **Elastic Disaster Recovery (DRS)**   | Continuously replicates servers into AWS for disaster recovery             |
| **Route 53 Failover Routing**         | Routes users to a healthy endpoint when the primary fails                  |
| **AWS DMS**                           | Migrates and can continuously replicate data between databases             |

---

# Important AWS DR services

## Aurora Global Database

Used when you need a **global relational database**.

```text
Region A
Aurora Primary
     │
     │ cross-region replication
     ↓
Region B
Aurora Secondary
```

Key idea:

> **Very low-latency cross-region database replication**

Exam association:

**"Aurora + global + cross-region DR"**

→ **Aurora Global Database**

---

## DynamoDB Global Tables

Used when you need:

* Multi-region DynamoDB
* Multi-active architecture
* Applications reading/writing in multiple regions

```text
Region A
DynamoDB
   ↕
Region B
DynamoDB
```

Exam association:

> **"Global, multi-region, multi-active NoSQL"**

→ **DynamoDB Global Tables**

---

## S3 Cross-Region Replication

Copies objects from one S3 bucket to another bucket in another AWS Region.

```text
S3 Region A
    │
    │ CRR
    ↓
S3 Region B
```

Useful for:

* Disaster recovery
* Compliance
* Keeping a regional copy of data

Exam association:

> **"Replicate S3 objects to another region"**

→ **S3 Cross-Region Replication**

---

## AWS Backup

AWS Backup provides a **central place to configure and manage backups** for supported AWS services.

Examples include:

* EBS
* RDS
* DynamoDB
* EFS

It is especially useful when the requirement says:

> "Centrally manage backups across multiple AWS services/accounts."

### Vault Lock

**Vault Lock** helps protect backups from being deleted or modified according to the configured retention policy.

Exam association:

> **"Centralized backups across AWS services"**

→ **AWS Backup**

> **"Backups must be protected from accidental or malicious deletion"**

→ **AWS Backup + Vault Lock**

---

## Elastic Disaster Recovery (DRS)

DRS is designed for **server disaster recovery**.

It continuously replicates server data into AWS so that the servers can be recovered when needed.

Typical scenario:

> "A company has hundreds of on-premises servers and wants cost-effective DR in AWS."

→ **Elastic Disaster Recovery (DRS)**

Memory:

> **DRS = replicate servers to AWS for disaster recovery**

---

## Route 53 Failover Routing

Route 53 can use health checks to detect that the primary endpoint is unhealthy and direct DNS traffic to the standby endpoint.

```text
                 Health check
                      ↓
Users → Route 53 → Primary
              ↘
               → DR
```

Exam association:

> **"Automatically send users to the DR region when the primary fails."**

→ **Route 53 Failover Routing**

---

## AWS DMS

AWS Database Migration Service is primarily for **database migration and replication**.

It can continuously replicate data from one database to another.

Think:

> **DMS = database movement / replication**

Do not confuse it with:

* **AWS Backup** → backups
* **Aurora Global Database** → Aurora cross-region database replication
* **DRS** → server replication

---

# The classic traps

## Trap 1 — "Best" does not mean "correct"

Suppose the requirement says:

> "The application can tolerate several hours of downtime and cost must be minimized."

You might think:

> "Active-Active has the best availability."

True, but irrelevant.

It costs much more than necessary.

→ **Backup & Restore**

---

## Trap 2 — Pilot Light vs Warm Standby

This is one of the most important differences.

### Pilot Light

```text
DR:
Database       ✅
Application    ❌ / minimal
```

### Warm Standby

```text
DR:
Database       ✅
Application    ✅
Everything     ✅
Capacity       ↓ reduced
```

So:

> **DB ready, compute mostly off → Pilot Light**

> **Whole environment running at smaller capacity → Warm Standby**

---

## Trap 3 — RPO and RTO are different

Do not mix them up.

### RPO

**How much data can we lose?**

```text
Last good data ──────── Disaster
       ◀──── data loss ────▶
              RPO
```

### RTO

**How long can we be down?**

```text
Disaster ─────────── Recovery
          ◀─ downtime ─▶
                 RTO
```

A system can have:

* excellent RPO but poor RTO
* excellent RTO but poor RPO

They are independent requirements.

---

# Pocket card

| Question / keyword                              | Think                         |
| ----------------------------------------------- | ----------------------------- |
| **Maximum acceptable data loss**                | **RPO**                       |
| **Maximum acceptable downtime**                 | **RTO**                       |
| **Cheapest DR, hours are acceptable**           | **Backup & Restore**          |
| **Database replicated, compute mostly off**     | **Pilot Light**               |
| **Smaller complete environment always running** | **Warm Standby**              |
| **Both regions actively serve traffic**         | **Multi-Site Active-Active**  |
| **Aurora cross-region relational DR**           | **Aurora Global Database**    |
| **Multi-region, multi-active NoSQL**            | **DynamoDB Global Tables**    |
| **Replicate S3 objects to another region**      | **S3 CRR**                    |
| **Centralized backups for many AWS services**   | **AWS Backup**                |
| **Protect backups from deletion/modification**  | **Vault Lock**                |
| **Replicate servers for DR**                    | **AWS DRS**                   |
| **DNS failover to another region**              | **Route 53 Failover Routing** |
| **Continuous database migration/replication**   | **AWS DMS**                   |

# Final mental model

When you see a DR question, think:

```text
                    DISASTER RECOVERY
                           │
             ┌─────────────┴─────────────┐
             │                           │
            RPO                         RTO
      "How much data              "How much downtime
       can we lose?"                can we tolerate?"
             │                           │
             └─────────────┬─────────────┘
                           ↓
                 Choose a DR strategy
                           │
        ┌──────────────────┼──────────────────┐
        ↓                  ↓                  ↓
   Backup & Restore   Pilot Light       Warm Standby
        │                  │                  │
     cheapest          core ready        full copy,
     rebuild          compute starts     smaller size
        │                  │                  │
        └──────────────────┴──────────────────┘
                           ↓
                  Active-Active
                  both regions live
```

### One sentence to remember

> **The tighter the RPO/RTO requirement, the more infrastructure you usually need running before the disaster.**

And for the exam:

> **First satisfy RPO/RTO → then choose the cheapest architecture that satisfies them.**
