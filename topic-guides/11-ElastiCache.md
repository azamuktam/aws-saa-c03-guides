# Section 11: ElastiCache

## The idea

**Amazon ElastiCache** is a managed in-memory data store used mainly to:

* Reduce database load
* Reduce application latency
* Store frequently accessed data
* Store shared application sessions
* Support real-time data structures such as leaderboards

Because data is held in memory, access can be much faster than reading from a disk-based database.

The application normally needs to be designed to use the cache:

```text
Application
     ↓
Check cache
     │
     ├── Hit → return cached data
     │
     └── Miss
          ↓
        Database
          ↓
      Store in cache
          ↓
      Return data
```

### Important distinction

**ElastiCache requires application-level cache logic.**

If the question says:

> "Speed up DynamoDB without changing application code."

→ **DAX**, not ElastiCache.

---

# Redis vs Memcached

For SAA, know the major differences.

| Feature                     | **Redis**                       | **Memcached** |
| --------------------------- | ------------------------------- | ------------- |
| Persistence                 | ✅                               | ❌             |
| Replication                 | ✅                               | ❌             |
| Multi-AZ automatic failover | ✅                               | ❌             |
| Backup and restore          | ✅                               | ❌             |
| Sorted sets                 | ✅                               | ❌             |
| Pub/Sub                     | ✅                               | ❌             |
| Multi-threaded architecture | Limited compared with Memcached | ✅             |
| Simplicity                  | More features                   | Simpler       |

### Redis

Choose Redis when the question requires features such as:

* Replication
* Failover
* Backup/restore
* Persistence
* Sorted sets
* Pub/Sub
* Shared sessions when Redis features/HA are required
* **Global Datastore** → cross-Region replication for Redis data

### Memcached

Choose Memcached when the requirement is essentially:

* Simple cache
* Very simple architecture
* Multi-threaded performance
* Data loss is acceptable
* No persistence or replication required
* **Auto Discovery** of cache nodes

### SAA shortcut

```text
Need advanced cache features
        ↓
Redis

Need a simple distributed cache
        ↓
Memcached
```

---

# Redis Authentication and Encryption

For Redis, keep these concepts separate:

| Requirement                              | Feature                     |
| ---------------------------------------- | --------------------------- |
| Encrypt traffic between client and Redis | **In-transit encryption**   |
| Encrypt Redis data at rest               | **At-rest encryption**      |
| Require a Redis password                 | **Redis AUTH / auth token** |
| AWS IAM-based authentication             | **IAM authentication**      |

### Redis AUTH

Redis AUTH requires clients to provide an authentication credential before issuing commands.

For an ElastiCache Redis deployment that requires a password, use an **AUTH token**.

Typical architecture:

```text
Client
  ↓
Redis AUTH
  ↓
ElastiCache for Redis
```

### Encryption in transit

Protects network traffic:

```text
Client
  ↓ TLS
ElastiCache Redis
```

This is **not authentication**.

### Encryption at rest

Protects data stored by the cache.

This is also **not authentication**.

### Exam pattern

> "Administrators must authenticate before executing Redis commands."

→ **Redis AUTH / auth token**

> "Redis traffic must be encrypted."

→ **In-transit encryption**

> "Cached data must be encrypted on storage."

→ **At-rest encryption**

---

# Caching strategies

The exam commonly tests three ideas.

## Lazy loading

Also called **cache-aside**.

The application reads from the cache first.

If the item is missing:

```text
Cache miss
   ↓
Database
   ↓
Return data
   ↓
Store in cache
```

Advantages:

* Only frequently requested data enters the cache.
* Avoids caching data that nobody requests.

Disadvantages:

* First request after a miss is slower.
* Cached data can become stale.

### Exam signal

> "Cache only data that users actually request."

→ **Lazy loading**

---

## Write-through

The application updates the cache whenever it updates the database.

```text
Application
   ├──► Database
   └──► Cache
```

Advantages:

* Cache is updated whenever the database changes.
* Reduces stale-cache problems.

Disadvantages:

* Every write also updates the cache.
* Data may be cached even when nobody reads it.

### Exam signal

> "Cached data must stay synchronized with database updates."

→ **Write-through**

---

## TTL

**TTL (Time To Live)** defines how long cached data remains valid.

Example:

```text
Price cached
   ↓
TTL = 60 seconds
   ↓
Entry expires
```

TTL is useful when some staleness is acceptable and you want old values to expire automatically.

### Exam signal

> "Cached data should expire automatically after a certain period."

→ **TTL**

---

# Lazy loading vs write-through

| Requirement                                | Better choice     |
| ------------------------------------------ | ----------------- |
| Cache only data that is actually requested | **Lazy loading**  |
| Minimize stale cached data                 | **Write-through** |
| Automatically remove old values            | **TTL**           |
| First request can tolerate cache miss      | **Lazy loading**  |

### Common trap

> "Prices or inventory must never become stale."

→ Prefer **write-through** rather than relying only on lazy loading.

---

# ElastiCache for sessions

A common architecture is:

```text
Users
  ↓
ALB
  ↓
EC2 Auto Scaling Group
  ├── EC2
  ├── EC2
  └── EC2
  ↓
ElastiCache
```

The application stores session information in a shared cache instead of in an individual EC2 instance.

This makes the application servers **stateless**.

### Why?

Suppose the user's session is stored only on EC2-A.

If EC2-A is terminated:

```text
User session
     ↓
EC2-A terminated
     ↓
Session lost
```

With a shared cache:

```text
User
 ↓
ALB
 ↓
Any EC2 instance
 ↓
ElastiCache
 ↓
Shared session
```

Any healthy application server can retrieve the user's session.

### Exam signal

> "Users are logged out when EC2 instances are terminated or scaled in."

→ **Use a shared session store**, such as **ElastiCache Redis** or DynamoDB.

---

# ElastiCache for repeated reads

ElastiCache is useful when the same data is requested repeatedly.

Example:

```text
10,000 requests
       ↓
Same product data
       ↓
ElastiCache
```

Instead of repeatedly querying the database, the application can reuse the cached result.

### Exam signal

> "Popular pages repeatedly query the same database records."

→ **ElastiCache**

---

# ElastiCache vs Read Replicas

Both can reduce database pressure, but they solve different problems.

| Requirement                        | Best choice      |
| ---------------------------------- | ---------------- |
| Same data requested repeatedly     | **ElastiCache**  |
| Many different read queries        | **Read Replica** |
| Analytical/reporting workload      | **Read Replica** |
| Need to cache application sessions | **ElastiCache**  |

### Why?

A cache works best when many requests ask for the **same data**.

If every query is different:

```text
Query A
Query B
Query C
Query D
...
```

there may be little cache reuse.

A read replica can handle more read queries instead.

---

# ElastiCache vs DAX

This is an important SAA comparison.

|                       | ElastiCache                 | DAX                                                     |
| --------------------- | --------------------------- | ------------------------------------------------------- |
| Main use              | General application caching | DynamoDB caching                                        |
| Supported data source | Application-defined         | DynamoDB                                                |
| Application changes   | **Required**                | **Minimal/none** because DAX is DynamoDB API-compatible |
| Redis/Memcached       | Yes                         | No                                                      |
| Best signal           | Repeated data / sessions    | DynamoDB + low latency + no code changes                |

### Exam pattern

> "DynamoDB reads are slow and the application should not be modified."

→ **DAX**

> "Application needs a general-purpose cache."

→ **ElastiCache**

---

# Redis data structures

Redis provides useful data structures that often appear in SAA questions.

Important examples:

* Strings
* Lists
* Sets
* Sorted sets
* Hashes

## Sorted sets

Sorted sets maintain members ordered by score.

This makes Redis useful for:

* Gaming leaderboards
* Ranking systems
* Scores

### Exam signal

> "Real-time gaming leaderboard."

→ **Redis sorted sets**

---

# Redis Pub/Sub

Redis supports **publish/subscribe messaging**.

Example:

```text
Publisher
    ↓
 Redis
    ↓
Subscribers
```

Useful when applications need lightweight real-time messaging.

### Exam signal

> "Publish/subscribe through the caching layer."

→ **Redis Pub/Sub**

---

# High availability

For Redis workloads that require availability:

```text
Primary
   ↓
Replica
   ↓
Automatic failover
```

ElastiCache for Redis supports replication and Multi-AZ automatic failover.

For Redis data that must be replicated across AWS Regions, use **Global Datastore**.

Memcached does not provide the same replication/failover model or Redis Global Datastore capability.

### Exam signal

> "Cache must survive a node failure."

→ **Redis with replication / Multi-AZ**

---

# Memcached Auto Discovery

**Auto Discovery** is an important Memcached-specific exam clue.

It allows a Memcached client to **automatically discover the nodes in the cache cluster** rather than requiring the application to manually maintain the list of cache nodes.

Typical pattern:

```text
Application
     ↓
Memcached client
     ↓
Auto Discovery
     ↓
┌─────────┬─────────┬─────────┐
│ Node 1  │ Node 2  │ Node 3  │
└─────────┴─────────┴─────────┘
```

When the cluster topology changes, the client can discover the updated set of nodes.

### Important distinction

**Auto Discovery ≠ Auto Scaling**

* **Auto Discovery** → client automatically discovers cache nodes.
* **Auto Scaling** → capacity is automatically increased/decreased.

### Exam signal

> "Distributed cache with multithreaded performance and Auto Discovery."

→ **ElastiCache for Memcached with Auto Discovery**

### Very important trap

A question may mention:

> "Users are located in multiple AWS Regions."

That **does not automatically mean Redis Global Datastore**.

Look at the actual requirement.

If the question emphasizes:

* **Memcached**
* **Auto Discovery**
* **Multithreaded performance**
* Simple distributed cache
* Sub-millisecond latency

→ **Memcached with Auto Discovery**

If it emphasizes:

* **Redis**
* Cross-Region **replication of Redis data**
* Redis data must be available in multiple AWS Regions

→ **Redis Global Datastore**

---

# Redis Global Datastore

**Redis Global Datastore** is used when Redis data needs to be replicated across **multiple AWS Regions**.

This is especially relevant for globally distributed applications where users in different Regions need access to Redis data with low latency.

Typical pattern:

```text
AWS Region A
EC2 → ElastiCache Redis
             ↓
       Global Datastore
             ↓
AWS Region B
EC2 → ElastiCache Redis
```

Use it when the question explicitly requires:

* Multiple AWS Regions
* **Cross-Region Redis replication**
* Globally distributed users
* Redis data available in different Regions
* Low-latency access to Redis data in different Regions

### Critical exam distinction

> **Multiple AWS Regions ≠ automatically Redis Global Datastore.**

The fact that users are distributed across Regions is not enough.

The question must require **cross-Region replication of Redis data** or otherwise clearly point to Global Datastore.

For example:

> "Users are distributed across multiple AWS Regions and Redis session data must be replicated across Regions."

→ **Redis Global Datastore**

But:

> "A fleet of EC2 instances needs a shared distributed cache with multithreaded performance and Auto Discovery."

→ **Memcached with Auto Discovery**

### Exam pattern

> "Need cross-Region replication for Redis session data."

→ **Redis Global Datastore**

> "Need a simple distributed cache with Auto Discovery and multithreaded performance."

→ **Memcached with Auto Discovery**

---

# Persistence

Redis supports persistence mechanisms, while Memcached is fundamentally a volatile cache.

This matters when the question says:

> "Cached data should survive a restart."

→ **Redis**

If cached data can simply be reconstructed:

→ **Memcached** may be appropriate.

---

# Question patterns

> **"Repeated identical reads are overloading RDS."**
> → **ElastiCache**

> **"Users are logged out when EC2 instances are terminated."**
> → **Shared session store such as ElastiCache Redis or DynamoDB**

> **"Users are distributed across multiple AWS Regions and Redis session data must be replicated across Regions."**
> → **ElastiCache for Redis Global Datastore**

> **"Users are distributed across multiple AWS Regions and a simple cache must provide shared session storage, multithreaded performance, and Auto Discovery."**
> → **ElastiCache for Memcached with Auto Discovery**

> **"Need cross-Region replication for Redis session data."**
> → **Redis Global Datastore**

> **"Need a real-time gaming leaderboard."**
> → **Redis sorted sets**

> **"Need Pub/Sub messaging."**
> → **Redis**

> **"Cache must survive node failure."**
> → **Redis with replication / Multi-AZ**

> **"Simple cache; data loss is acceptable."**
> → **Memcached**

> **"Need a distributed cache with the fewest features possible."**
> → **Memcached**

> **"Distributed cache requires Auto Discovery."**
> → **Memcached with Auto Discovery**

> **"Need multithreaded cache performance."**
> → **Memcached**

> **"Cached values must stay synchronized with database writes."**
> → **Write-through**

> **"Only frequently requested objects should enter the cache."**
> → **Lazy loading**

> **"Cached data should expire automatically after a period."**
> → **TTL**

> **"DynamoDB needs faster reads without changing application code."**
> → **DAX**

> **"Users must authenticate before issuing Redis commands."**
> → **Redis AUTH / auth token**

> **"Redis traffic must be encrypted."**
> → **In-transit encryption**

> **"Redis data must be encrypted while stored."**
> → **At-rest encryption**

---

# Pocket card

| Keyword                               | Answer                                    |
| ------------------------------------- | ----------------------------------------- |
| Repeated identical reads              | **ElastiCache**                           |
| Shared application sessions           | **ElastiCache**                           |
| Cross-Region **Redis replication**    | **Redis Global Datastore**                |
| Multiple Regions alone                | **Not enough to choose Global Datastore** |
| Gaming leaderboard                    | **Redis sorted sets**                     |
| Pub/Sub                               | **Redis**                                 |
| Persistence                           | **Redis**                                 |
| Replication / failover                | **Redis**                                 |
| Simple cache, loss acceptable         | **Memcached**                             |
| Multi-threaded simple cache           | **Memcached**                             |
| Auto Discovery                        | **Memcached**                             |
| Auto Discovery + multithreaded cache  | **Memcached**                             |
| Cache only after a read miss          | **Lazy loading**                          |
| Keep cache updated on database writes | **Write-through**                         |
| Automatically expire cached data      | **TTL**                                   |
| Analytical/diverse reads              | **Read Replica**                          |
| DynamoDB + no application changes     | **DAX**                                   |
| Redis password authentication         | **AUTH token**                            |
| Encrypt Redis network traffic         | **In-transit encryption**                 |
| Encrypt Redis stored data             | **At-rest encryption**                    |
| Sub-millisecond cache access          | **In-memory cache**                       |

### Final mental model

```text
                    ElastiCache
                        │
             ┌──────────┴──────────┐
             │                     │
          Redis                 Memcached
             │                     │
     Advanced features       Simple cache
     Replication             Multithreaded
     Failover                Auto Discovery
     Persistence             Loss acceptable
     Sorted sets
     Pub/Sub
             │
      Global Datastore
             │
      Cross-Region Redis
       replication
```

**Redis = feature-rich / HA / persistence / replication**

**Memcached = simple / multithreaded / Auto Discovery**

**Global Datastore = specifically cross-Region Redis replication**

**DAX = DynamoDB-specific cache with minimal application changes**
