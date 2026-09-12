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
* Shared sessions

### Memcached

Choose Memcached when the requirement is essentially:

* Simple cache
* Very simple architecture
* Multi-threaded performance
* Data loss is acceptable
* No persistence or replication required

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
ElastiCache Redis
```

The application stores session information in Redis instead of in an individual EC2 instance.

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

With Redis:

```text
User
 ↓
ALB
 ↓
Any EC2 instance
 ↓
Redis
 ↓
Shared session
```

Any healthy application server can retrieve the user's session.

### Exam signal

> "Users are logged out when EC2 instances are terminated or scaled in."

→ **Store sessions in ElastiCache Redis** or another shared session store such as DynamoDB.

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

| Requirement                        | Best choice           |
| ---------------------------------- | --------------------- |
| Same data requested repeatedly     | **ElastiCache**       |
| Many different read queries        | **Read Replica**      |
| Analytical/reporting workload      | **Read Replica**      |
| Need to cache application sessions | **ElastiCache Redis** |

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

This is a strong Redis-specific clue.

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

Memcached does not provide the same replication/failover model.

### Exam signal

> "Cache must survive a node failure."

→ **Redis with replication / Multi-AZ**

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
> → **ElastiCache Redis for shared sessions** (or another shared session store)

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

| Keyword                               | Answer                    |
| ------------------------------------- | ------------------------- |
| Repeated identical reads              | **ElastiCache**           |
| Shared application sessions           | **Redis**                 |
| Gaming leaderboard                    | **Redis sorted sets**     |
| Pub/Sub                               | **Redis**                 |
| Persistence                           | **Redis**                 |
| Replication / failover                | **Redis**                 |
| Simple cache, loss acceptable         | **Memcached**             |
| Multi-threaded simple cache           | **Memcached**             |
| Cache only after a read miss          | **Lazy loading**          |
| Keep cache updated on database writes | **Write-through**         |
| Automatically expire cached data      | **TTL**                   |
| Analytical/diverse reads              | **Read Replica**          |
| DynamoDB + no application changes     | **DAX**                   |
| Redis password authentication         | **AUTH token**            |
| Encrypt Redis network traffic         | **In-transit encryption** |
| Encrypt Redis stored data             | **At-rest encryption**    |
| Microsecond-level cache access        | **In-memory cache**       |

---

# Core mental model

When you see an ElastiCache question, first identify the requirement:

```text
Repeated database reads
        ↓
ElastiCache

Shared sessions
        ↓
Redis

Leaderboard / ranking
        ↓
Redis sorted sets

Pub/Sub
        ↓
Redis

Simple cache + data loss acceptable
        ↓
Memcached

DynamoDB + no application changes
        ↓
DAX
```

Then identify the cache behavior:

```text
Read miss → populate cache
        ↓
Lazy loading

Database write → update cache
        ↓
Write-through

Automatically expire cached item
        ↓
TTL
```

And for Redis security:

```text
Require password
        ↓
AUTH token

Encrypt network traffic
        ↓
In-transit encryption

Encrypt stored cache data
        ↓
At-rest encryption
```

The most important SAA distinctions are:

**ElastiCache = general-purpose application caching.**

**Redis = advanced features, sessions, failover, sorted sets, Pub/Sub.**

**Memcached = simple distributed cache.**

**DAX = DynamoDB-specific caching with minimal application changes.**
