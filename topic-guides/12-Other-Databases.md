# Section 12: The Other Databases

## The idea

AWS has different database services because different applications need different **data models** and different types of queries.

The exam usually gives you the **workload or keyword**, and you choose the database that matches it.

The easiest approach is:

> **First identify what kind of data you have. Then identify how you need to query it.**

Examples:

```text
Need analytics on huge datasets
→ Redshift

Need SQL directly on files in S3
→ Athena

Need relationships between things
→ Neptune

Need time-based measurements
→ Timestream

Need MongoDB-compatible documents
→ DocumentDB

Need Cassandra-compatible wide-column storage
→ Keyspaces

Need full-text / fuzzy search
→ OpenSearch
```

---

## OLTP vs OLAP

This is important because it helps you distinguish **application databases** from **analytics databases**.

### OLTP — Online Transaction Processing

OLTP handles **many small, fast transactions**.

Examples:

```text
Create an order
Update a user's address
Check account balance
Insert a payment
```

Typical databases:

* RDS
* Aurora
* DynamoDB

The application usually works with a relatively small amount of data per request.

### OLAP — Online Analytical Processing

OLAP handles **large analytical queries**.

Example:

> "Calculate total revenue for every country for the last five years."

One query may scan millions or billions of rows.

Typical service:

**Amazon Redshift**

OLAP systems commonly use **columnar storage**, which is efficient when a query needs only a few columns from a huge dataset.

### THE trap

> *"Analysts run large reporting queries against the production application database and the application becomes slow."*

Don't keep running heavy analytics against the production OLTP database.

Move the analytics workload to something designed for it, such as:

* **Redshift**
* **Athena**, if the data is in S3
* a **Read Replica**, when appropriate for the scenario

---

## The database services

| Service                       | The identifying keyword                                    | What it is                                               |
| ----------------------------- | ---------------------------------------------------------- | -------------------------------------------------------- |
| **Redshift**                  | **Data warehouse / OLAP / BI**                             | Analytics warehouse for large datasets                   |
| **Athena**                    | **SQL on S3**                                              | Serverless SQL queries directly on data in S3            |
| **Neptune**                   | **Relationships / graph**                                  | Graph database                                           |
| **Timestream**                | **Time series**                                            | Database for timestamped data                            |
| **DocumentDB**                | **MongoDB**                                                | MongoDB-compatible document database                     |
| **Keyspaces**                 | **Cassandra**                                              | Managed Apache Cassandra-compatible wide-column database |
| **OpenSearch**                | **Full-text / fuzzy search / logs**                        | Search and log analytics service                         |
| **Amazon Managed Blockchain** | **Blockchain / multiple parties / decentralized networks** | Managed blockchain infrastructure and blockchain access  |
| **QLDB**                      | **Legacy — do not study as a current service**             | AWS ended support on **July 31, 2025**                   |

---

## Redshift

**Amazon Redshift = data warehouse for analytics.**

Use it when you have:

* OLAP workloads
* BI dashboards
* complex analytical queries
* very large datasets
* data warehouses

Example:

```text
S3 / databases / applications
          ↓
       Redshift
          ↓
   BI / analytics
```

Redshift uses **columnar storage**, which is efficient for analytical queries that read a small number of columns from many rows.

### Redshift Spectrum

**Redshift Spectrum** allows Redshift to query data that is **still stored in S3**.

You don't have to load all the S3 data into Redshift first.

```text
Redshift
   ↓
Redshift Spectrum
   ↓
S3 data
```

### Redshift Serverless

**Redshift Serverless** provides a serverless data warehouse.

You don't manage the Redshift cluster yourself.

Use it when:

> "We need a warehouse but don't want to manage cluster infrastructure."

### Remember

> **Warehouse / OLAP / BI → Redshift**

> **Query S3 from Redshift without loading it → Redshift Spectrum**

---

## Athena

**Amazon Athena = SQL directly on data stored in S3.**

You don't need to load the data into a database first.

```text
S3
 ↓
Athena
 ↓
SQL query
```

It is especially useful for:

* log analysis
* ad-hoc queries
* occasional analytics
* querying data already stored in S3

Athena is **serverless** and charges based on the amount of data scanned. The standard pricing model is **$5 per TB scanned**.

### Athena vs Redshift

```text
Data stays in S3
+ occasional/ad-hoc SQL
+ no infrastructure
→ ATHENA

Data is used as a warehouse
+ repeated analytics
+ complex BI queries
→ REDSHIFT
```

### Redshift Spectrum

```text
Already using Redshift
+ need to query data still in S3
→ Redshift Spectrum
```

---

## Athena cost optimization

Athena charges based on **data scanned**, so the goal is:

> **Scan less data.**

Three important techniques:

### 1. Use columnar formats

Use:

* **Parquet**
* **ORC**

Columnar formats allow Athena to read only the columns needed by the query.

### 2. Compress the data

Compression reduces the amount of data that needs to be read.

### 3. Partition the data

For example:

```text
S3
└── year=2026
    ├── month=01
    ├── month=02
    └── month=03
```

If the query only needs March, Athena can avoid scanning unrelated partitions.

### Memory

> **Athena cost = data scanned**

> **Reduce scanned data = partition + compress + Parquet/ORC**

AWS documents these optimizations and their impact on Athena query cost and performance.

---

## Neptune

**Amazon Neptune = graph database.**

Use it when the important part of the data is the **relationships between things**.

Examples:

```text
Person → friends → Person
Customer → bought → Product
Account → transferred money to → Account
Person → works for → Company
```

Typical workloads:

* social networks
* recommendation engines
* fraud detection
* knowledge graphs
* relationship-heavy applications

### Example

> "Find friends of friends."

That's a graph query.

→ **Neptune**

### Remember

> **Relationships are the main data → Neptune**

---

## Amazon Timestream

**Amazon Timestream = time-series database.**

Use it when data arrives with timestamps and you frequently query it by time.

Examples:

```text
Temperature at 10:01
Temperature at 10:02
Temperature at 10:03
```

Typical workloads:

* IoT sensors
* application metrics
* infrastructure monitoring
* telemetry
* industrial equipment data
* time-based measurements

### Example

> "Store millions of sensor readings and analyze them by hour, day, or month."

→ **Timestream**

### Remember

> **Timestamped measurements → Timestream**

---

## DocumentDB

**Amazon DocumentDB = MongoDB-compatible document database.**

It stores document-style data such as JSON-like documents.

Example:

```text
{
  "name": "Azam",
  "skills": ["PHP", "AWS"],
  "location": "Tashkent"
}
```

Typical use cases:

* document-oriented applications
* MongoDB-compatible workloads
* migrating MongoDB applications to a managed AWS service

### Example

> "The company has a MongoDB workload and wants a managed AWS database."

→ **DocumentDB**

### Remember

> **MongoDB → DocumentDB**

Important:

> DocumentDB is **MongoDB-compatible**, not literally MongoDB.

---

## Keyspaces

**Amazon Keyspaces = managed Apache Cassandra-compatible database.**

Cassandra is a **wide-column NoSQL database** designed for:

* very high throughput
* large amounts of data
* high availability
* predictable low latency

Amazon Keyspaces is **serverless** and automatically scales the tables based on traffic.

### Example

> "The company has an existing Cassandra workload and wants a managed AWS service."

→ **Amazon Keyspaces**

### Remember

> **Cassandra → Keyspaces**

---

## OpenSearch

**Amazon OpenSearch Service = search and log analytics.**

It is designed for searching large amounts of text/data quickly.

Typical workloads:

* full-text search
* fuzzy/typo-tolerant search
* log analysis
* application search
* dashboards and observability

### Full-text search

Suppose a customer searches:

```text
"running shoes"
```

and types:

```text
"runing shoes"
```

OpenSearch can provide fuzzy/full-text search behavior.

### Example

> "Product catalog needs typo-tolerant full-text search."

→ **OpenSearch**

### Log analytics

OpenSearch can also be used to analyze logs and build dashboards.

### Remember

> **Search / fuzzy search / log analytics → OpenSearch**

---

## DynamoDB + OpenSearch

DynamoDB is excellent for key-value/document access, but it is **not a general full-text/fuzzy search engine**.

A common architecture is:

```text
DynamoDB
    ↓
DynamoDB Streams
    ↓
Lambda
    ↓
OpenSearch
```

The application stores the main data in DynamoDB.

The stream captures changes.

Lambda sends those changes to OpenSearch.

Search queries go to OpenSearch.

### Example

> "Product data is stored in DynamoDB, but customers need fuzzy full-text search."

→ **DynamoDB + Streams + Lambda + OpenSearch**

---

## Amazon Managed Blockchain

Amazon Managed Blockchain provides managed infrastructure and APIs for blockchain networks.

Current AWS documentation describes support for blockchain frameworks including **Hyperledger Fabric and Ethereum**, while AMB Access also provides access to public blockchain networks such as Ethereum and Bitcoin.

The key idea for the exam is:

> **Blockchain is for multiple parties that need shared, verifiable records without relying on one central database owner.**

### Example

Several organizations need to share transaction records but don't want one company to control the entire ledger.

→ **Managed Blockchain**

### Remember

> **Multiple parties + blockchain → Managed Blockchain**

---

## QLDB — important update

**Amazon QLDB is no longer a current AWS service.**

AWS ended support for QLDB on **July 31, 2025**. AWS provides migration guidance toward **Aurora PostgreSQL**.

Therefore, for your current SAA notes:

> **Do not memorize QLDB as a current service.**

You may see old study material mentioning:

> "Immutable, cryptographically verifiable centralized ledger"

That was **QLDB**, but it is now a legacy/outdated service.

---

## Redshift vs Athena

Both can perform SQL analytics, but they are used differently.

```text
Data already in S3
+ occasional/ad-hoc queries
+ no infrastructure
→ ATHENA

Data warehouse
+ repeated analytics
+ complex queries
+ BI dashboards
→ REDSHIFT
```

### Middle case

```text
Data remains in S3
+ already using Redshift
→ REDSHIFT SPECTRUM
```

---

## Athena vs OpenSearch

These are sometimes confused because both can analyze data.

```text
Need SQL analytics on files in S3
→ Athena

Need fast text search / fuzzy search
→ OpenSearch
```

Example:

> "Analyze ALB logs stored in S3."

→ **Athena**

Example:

> "Search millions of product descriptions with typo tolerance."

→ **OpenSearch**

---

## Neptune vs DynamoDB

```text
Simple key-value/document access
→ DynamoDB

Complex relationships between entities
→ Neptune
```

Example:

> "Get customer order by customer ID."

→ **DynamoDB**

Example:

> "Find relationships between accounts in a fraud network."

→ **Neptune**

---

## The decision algorithm

When you see an "other database" question, first identify the **workload**.

```text
Analytics / data warehouse
→ Redshift

SQL directly on S3
→ Athena

Search / fuzzy search / logs
→ OpenSearch

Relationships / graph
→ Neptune

Time-series / IoT / metrics
→ Timestream

MongoDB-compatible documents
→ DocumentDB

Cassandra-compatible wide-column
→ Keyspaces

Blockchain / multiple parties / shared ledger
→ Managed Blockchain
```

---

## Question patterns

> *"BI team needs a large data warehouse for complex analytical queries"* → **Redshift**

> *"Query data in S3 using SQL without managing servers"* → **Athena**

> *"Already using Redshift but need to query files that remain in S3"* → **Redshift Spectrum**

> *"Reduce Athena costs and improve query performance"* → **Parquet/ORC + compression + partitioning**

> *"Social network needs friend-of-friend queries"* → **Neptune**

> *"Detect fraud based on relationships between accounts"* → **Neptune**

> *"Millions of IoT readings need to be analyzed by time window"* → **Timestream**

> *"Migrate a MongoDB workload to a managed AWS service"* → **DocumentDB**

> *"Migrate a Cassandra workload to a managed AWS service"* → **Keyspaces**

> *"Product catalog needs fuzzy and typo-tolerant full-text search"* → **OpenSearch**

> *"DynamoDB data needs full-text search"* → **DynamoDB Streams → Lambda → OpenSearch**

> *"Several companies need to maintain shared blockchain records"* → **Amazon Managed Blockchain**

> *"Analytical queries are slowing down the production OLTP database"* → **Move the analytical workload away from the production database**, typically to **Redshift** or **Athena** depending on where the data is and the workload.

---

## Pocket card

| Keyword                                      | Answer                                               |
| -------------------------------------------- | ---------------------------------------------------- |
| Data warehouse / OLAP / BI / large analytics | **Redshift**                                         |
| Query S3 with SQL / serverless / ad-hoc      | **Athena**                                           |
| Query S3 from Redshift without loading       | **Redshift Spectrum**                                |
| Reduce Athena cost                           | **Parquet/ORC + compression + partitioning**         |
| Graph / relationships / social / fraud       | **Neptune**                                          |
| Time-series / IoT / telemetry / metrics      | **Timestream**                                       |
| MongoDB-compatible                           | **DocumentDB**                                       |
| Cassandra-compatible                         | **Keyspaces**                                        |
| Full-text / fuzzy search                     | **OpenSearch**                                       |
| Log analytics / search dashboards            | **OpenSearch**                                       |
| DynamoDB + full-text search                  | **DynamoDB Streams → Lambda → OpenSearch**           |
| Multiple parties + blockchain                | **Managed Blockchain**                               |
| QLDB                                         | **Legacy — ended support July 31, 2025**             |
| Analytics slowing production OLTP            | **Move analytics to Redshift/Athena as appropriate** |

## Final memory

```text
Redshift
= BIG ANALYTICS

Athena
= SQL ON S3

Spectrum
= REDSHIFT → S3 DATA

Neptune
= RELATIONSHIPS

Timestream
= TIME

DocumentDB
= MONGODB

Keyspaces
= CASSANDRA

OpenSearch
= SEARCH

Managed Blockchain
= SHARED BLOCKCHAIN
```

The most important distinction is:

```text
Need to run the application
→ RDS / Aurora / DynamoDB

Need to analyze huge datasets
→ Redshift / Athena

Need to search text
→ OpenSearch

Need to understand relationships
→ Neptune

Need timestamped measurements
→ Timestream
```
