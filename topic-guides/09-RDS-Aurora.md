# Section 9: RDS & Aurora

## The idea

Running your own database server means patching the OS, applying engine updates, taking backups at 2 a.m., and rebuilding everything when the hardware dies. **RDS (Relational Database Service) = AWS runs the database for you.** You pick an engine — **MySQL, PostgreSQL, MariaDB, Oracle, or SQL Server** — and AWS handles the machinery underneath.

The price of that convenience: **you get NO OS access**. You can't SSH in, can't install custom extensions at the OS level, can't tweak the engine binaries. If a question demands **custom engine configuration or OS-level access**, RDS is out — the answer is **database on EC2** (full control, all the toil) or **RDS Custom** (a halfway house for Oracle/SQL Server).

Now for **THE core distinction — the most-tested database fact on the entire exam.** RDS has two features that both involve "extra copies of your database," and they exist for completely different reasons. Think of your car:

- **Multi-AZ is the spare tire.** It exists for the day something goes wrong. You never drive on it during normal life.
- **Read Replicas are extra checkout lanes at the supermarket.** They exist to serve more customers at once, every ordinary day.

| | **Multi-AZ** (availability) | **Read Replicas** (performance) |
|---|---|---|
| Replication | **Synchronous** (standby is always exactly current) | **Asynchronous** (slight lag) |
| Where | Standby in **another AZ** | **Same AZ, cross-AZ, or cross-REGION** |
| How many | 1 standby | **Up to 15** |
| Can you read from it? | **NO — ZERO reads from the standby** | **Yes — that's the whole point** (read-only endpoints) |
| Failover | **Automatic, 60–120 seconds**, via **DNS flip** (same endpoint, new machine behind it) | **None automatic — manual promotion only** |
| Solves | AZ failure, hardware death | Read-heavy workloads, reporting |

**THE trap:** *"use the Multi-AZ standby to serve read traffic"* → **impossible**. The standby is invisible until failover — the spare tire stays in the trunk.

**THE trap (mirror image):** *"read replica provides automatic failover"* → **no**. Promotion is a **manual** decision — which also makes a cross-region replica a decent **budget DR** plan, just not an automatic one.

And note: **production uses BOTH** — Multi-AZ for surviving failures, replicas for scaling reads. They're not competitors.

## Backups & encryption

- **Automated backups** → enable **PITR (Point-In-Time Recovery)**, retention **maximum 35 days**. Deleted when the instance is deleted.
- **Manual snapshots** → kept **forever**, until you delete them. **"Retain backups for 5 years / 7 years / compliance" → manual snapshot**, always — 35 days is the wall automated backups can't cross.

**Encryption is a birth decision.** You can only encrypt an RDS database **at creation**. To encrypt an existing unencrypted database, do the snapshot dance (this question is essentially guaranteed):

```
unencrypted DB → snapshot → COPY the snapshot (enable encryption) → restore from encrypted copy → repoint the app
```

You cannot flip encryption on in place, and you cannot encrypt the original snapshot directly — you encrypt the **copy**.

## RDS Proxy

Databases hate being swarmed. Every connection costs memory, and **Lambda** — which can scale to thousands of concurrent executions in seconds — will happily open thousands of connections and knock your database over. **RDS Proxy = a connection pool** that sits in front of RDS, letting thousands of clients share a small, warm set of database connections.

- **Lambda + RDS = RDS Proxy. Always.** If those two appear in the same sentence with "too many connections" or "connection errors," you're done reading.
- Bonus: the proxy also **shrinks failover time** (it holds client connections and re-routes them, instead of everyone re-resolving DNS).

Also worth one neuron: **storage autoscaling** — RDS can grow its storage automatically when it runs low ("database keeps running out of disk with unpredictable growth" → enable storage autoscaling).

## Aurora

Aurora is AWS's own cloud-native engine, **compatible with MySQL and PostgreSQL** (your app connects the same way). The architectural trick: Aurora **separates compute from storage**.

```
   [Writer node]  [Reader]  [Reader] ...   ← compute: disposable, pluggable
        │            │         │
   ═════╧════════════╧═════════╧═════════
     Shared storage: 6 copies across 3 AZs, self-healing,
     auto-grows to 128 TB
```

Because every node plugs into the **same shared storage**, a failed writer is replaced by simply promoting a reader that already sees all the data:

- **Failover in under 30 seconds** (vs 60–120s for RDS Multi-AZ)
- **Up to 15 replicas with ~millisecond lag** (vs async seconds on RDS)
- **Storage auto-grows to 128 TB** — no provisioning
- **Two endpoints**: the **writer endpoint** (points at the current writer, survives failover) and the **reader endpoint** (load-balances across all replicas). Apps send writes to one and reads to the other — never hardcode instance addresses.

**Aurora's special features — feature → scenario:**

| Feature | The scenario it answers |
|---|---|
| **Serverless v2** | **Spiky, unpredictable, or idle** workloads — dev/test databases used a few hours a day, capacity scales itself, pay for what you use |
| **Global Database** | **Cross-region DR with RPO ~1 second, RTO <1 minute**, plus local low-latency reads on other continents. (A plain cross-region read replica = slower replication, slower promotion — the budget distractor) |
| **Cloning** | **Copy-on-write instant copy** of a production database — a full-size staging/test copy in minutes without duplicating storage |
| **Backtrack** | **Rewind the database in place** (Aurora MySQL) — *"undo an accidental delete/bad migration FAST, without restoring from backup"*, no new instance, no restore wait |

RPO (Recovery Point Objective) = how much data you may lose; RTO (Recovery Time Objective) = how long until you're back. Aurora Global Database's "~1s / <1min" pair is a memorize-it number.

## Question patterns

> *"Database must survive an AZ failure with no manual intervention"* → **Multi-AZ** ("survive failure" + "automatic" = spare tire, 60–120s DNS flip).

> *"Reporting/analytics queries are slowing down the production database"* → **Read Replica** (offload reads to a checkout lane — but if the queries are the *same ones repeatedly*, **ElastiCache** wins; replica is for **diverse/complex** queries).

> *"Lambda functions are exhausting database connections"* → **RDS Proxy** (Lambda + RDS = Proxy, every time).

> *"Encrypt an existing unencrypted RDS database"* → **snapshot → copy with encryption → restore** (encryption is creation-time only; this is the snapshot dance).

> *"Global application needs cross-region DR with less than 1 minute recovery"* → **Aurora Global Database** (RPO ~1s, RTO <1min — nothing else touches it).

> *"Dev database sits idle nights and weekends; minimize cost"* → **Aurora Serverless v2** (spiky/idle/unpredictable = serverless).

> *"Need a full copy of the production database for testing, quickly and cheaply"* → **Aurora Cloning** (copy-on-write = instant, near-free until it diverges).

> *"Developer ran a bad DELETE; restore the database to 5 minutes ago as fast as possible"* → **Aurora Backtrack** (rewind in place — no restore, no new instance).

> *"Compliance requires database backups kept for 7 years"* → **Manual snapshots** (automated backups cap at 35 days).

> *"Application needs OS-level access / a custom database engine"* → **EC2** (or RDS Custom for Oracle/SQL Server) — plain RDS gives no OS access.

## Pocket card

| Keyword in question | Answer |
|---|---|
| survive AZ failure, auto-failover | **Multi-AZ** (sync, 60–120s, DNS flip) |
| standby serves reads | **TRAP — never** |
| scale reads, offload reporting | **Read Replica** (async, up to 15, manual promotion) |
| replica auto-failover | **TRAP — promotion is manual** |
| same queries over and over | ElastiCache (not a replica) |
| Lambda + RDS connections | **RDS Proxy** |
| backups > 35 days / "retain X years" | **Manual snapshot** |
| restore to any point in time (≤35 days) | Automated backups / PITR |
| encrypt existing DB | **Snapshot → copy-encrypted → restore** |
| OS access / custom engine | EC2 or RDS Custom |
| failover <30s, 15 low-lag replicas, 128TB autogrow | **Aurora** |
| spiky / idle / unpredictable load | **Aurora Serverless v2** |
| cross-region DR, RPO ~1s, RTO <1min | **Aurora Global Database** |
| instant prod copy for staging | **Aurora Cloning** |
| undo mistake fast, no restore | **Aurora Backtrack** |

You now know how to keep the database standing and fast — but when the exam's read traffic is the *same* hot data thousands of times per second, even fifteen replicas is the wrong answer, and that's where caching with ElastiCache takes over.
