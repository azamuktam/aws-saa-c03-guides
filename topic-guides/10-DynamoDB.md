# Section 10: DynamoDB

## The idea

DynamoDB is AWS's **serverless NoSQL database**.

* **Serverless** = no servers to provision, patch, size, or maintain. You use APIs.
* **NoSQL** = data is stored as **items** with flexible attributes. No joins or rigid relational schema.
* Designed for **single-digit millisecond latency at scale**.

### DynamoDB vs RDS — the first fork in every question

| Scenario says...                                                                        | Pick             |
| --------------------------------------------------------------------------------------- | ---------------- |
| Joins, complex queries, existing SQL app, relational schema                             | **RDS / Aurora** |
| Massive scale, serverless, key-value lookups, millisecond latency, unpredictable growth | **DynamoDB**     |

**THE trap:** *"migrate with no code changes from MySQL"* → **RDS/Aurora**, not DynamoDB. Moving to NoSQL requires query/data-model changes.

### Keys and hot partitions

Every table has a **partition key** and may also have a **sort key**.

* **Partition key** → determines partition placement.
* **Sort key** → orders items within the same partition key and allows multiple related items.
* DynamoDB hashes the partition key to distribute items across partitions.

**Hot partition:** too much traffic targets the same partition key.

Example:

```text
Good key: user_id                 Bad key: country
[P1][P2][P3][P4]                 [P1][P2][P3][P4]
 ▲▲  ▲▲  ▲▲  ▲▲                  ████  .   .   .
 even spread                      hot!  idle idle idle
```

**Rule:** prefer **high-cardinality** partition keys such as `user_id` or `order_id`; avoid low-cardinality keys such as `status`, `country`, or `date`.

### Capacity modes — same logic as EC2 pricing

* **Provisioned**

  * Specify read/write capacity.
  * Best for **steady, predictable traffic**.
  * Can use **auto scaling** to adjust capacity within limits.

* **On-Demand**

  * Pay per request.
  * No capacity planning.
  * Best for **spiky, unpredictable, or new workloads**.

**THE trap:** *"app is throttled by unpredictable traffic spikes"* → **On-Demand mode**, rather than simply over-provisioning.

### RCU / WCU — the exam arithmetic

| Unit      | Capacity                                                                                            |
| --------- | --------------------------------------------------------------------------------------------------- |
| **1 RCU** | **1 strongly consistent read/sec** for an item ≤ **4 KB**, or **2 eventually consistent reads/sec** |
| **1 WCU** | **1 write/sec** for an item ≤ **1 KB**                                                              |

* **Eventually consistent** = may return a slightly older value and uses half the read capacity.
* **Strongly consistent** = reads the latest write.
* Default read consistency is **eventual**.

**THE trap:** *"must always read the latest write"* → **Strongly consistent read**.

### Feature zoo → scenario matcher

| Feature                           | One-liner                                                                                                                                                                                                                        |
| --------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **DAX** (DynamoDB Accelerator)    | In-memory DynamoDB cache for **microsecond reads**; **API-compatible**, so applications can use it with minimal/no code changes. **DynamoDB only.** For other services or more flexible caching → **ElastiCache**.               |
| **Global Tables**                 | **Multi-region active-active** DynamoDB. Users can read and write locally in multiple regions; DynamoDB replicates data across regions.                                                                                          |
| **Streams**                       | **24-hour change feed** for item inserts, updates, and deletes; commonly triggers **Lambda**. Keyword: *"react to item changes."*                                                                                                |
| **TTL** (Time to Live)            | Set an expiry timestamp attribute; DynamoDB automatically deletes expired items. Deletion is free for the normal TTL deletion path and can occur within about **48 hours** after expiry. Common for sessions and temporary data. |
| **GSI** (Global Secondary Index)  | Query using a **different partition/sort key** from the base table. Can be added **anytime** and has **its own capacity**.                                                                                                       |
| **LSI** (Local Secondary Index)   | Uses the **same partition key** with a different sort key. Must be created **when the table is created**.                                                                                                                        |
| **Transactions**                  | **ACID**, all-or-nothing operations across multiple items/tables. Uses **2× capacity**.                                                                                                                                          |
| **PITR** (Point-In-Time Recovery) | Continuous backup; restore to any point in the previous **35 days**.                                                                                                                                                             |

**THE index trap:**

* Existing table + need another access pattern → **GSI**
* New table + same partition key but different sort key → **LSI**

### Two more exam staples

**Serverless web/API stack:**

```text
User → CloudFront → API Gateway → Lambda → DynamoDB
        (CDN)       (front door)   (code)    (data)
```

Typical answer for a **fully serverless web/API architecture**.

**400 KB item limit**

* Maximum DynamoDB item size = **400 KB**.
* For images/documents larger than this:

  * Store the object in **S3**
  * Store the **S3 key/pointer** in DynamoDB.

## Question patterns

> *"Flash sale causes unpredictable spikes; table throttles"* → **On-Demand capacity mode**

> *"Reduce read latency to microseconds without changing application code"* → **DAX**

> *"Global users need low-latency reads AND writes in every region"* → **Global Tables** (active-active)

> *"Run custom logic whenever items are added or modified"* → **DynamoDB Streams + Lambda**

> *"Automatically remove session records after 30 minutes, at no cost"* → **TTL** (automatic deletion; ~48-hour deletion window)

> *"Table keyed on user_id, but app must also query by email"* → **GSI**

> *"Debit one account and credit another — both or neither"* → **DynamoDB Transactions** (ACID; 2× capacity)

> *"Store 2 MB documents per record"* → **S3 + DynamoDB pointer** (400 KB item limit)

> *"Application must always see the most recent write"* → **Strongly consistent read**

> *"Reporting needs joins across many tables"* → **RDS/Aurora**

## Pocket card

| Keyword                                                     | Answer                                  |
| ----------------------------------------------------------- | --------------------------------------- |
| Serverless NoSQL, key-value, any scale                      | **DynamoDB**                            |
| Joins / complex SQL                                         | **RDS / Aurora**                        |
| Spiky / unknown traffic                                     | **On-Demand mode**                      |
| Steady, predictable traffic                                 | **Provisioned + auto scaling**          |
| Microseconds, DynamoDB cache                                | **DAX**                                 |
| Multi-region active-active                                  | **Global Tables**                       |
| React to item changes                                       | **Streams → Lambda**                    |
| Auto-expire items                                           | **TTL**                                 |
| Query by another attribute on an existing table             | **GSI**                                 |
| Alternate sort key, same partition key, table creation only | **LSI**                                 |
| Atomic multi-item operation                                 | **Transactions**                        |
| Restore to any point in last 35 days                        | **PITR**                                |
| Item > 400 KB                                               | **S3 + pointer**                        |
| 1 RCU                                                       | **1 strong or 2 eventual reads ≤ 4 KB** |
| 1 WCU                                                       | **1 write ≤ 1 KB**                      |

**Final distinction:** DynamoDB is for scalable NoSQL access patterns; when you need relational features such as joins and complex SQL, use **RDS/Aurora**. When you need an in-memory caching layer, see **ElastiCache (Section 11)**.
