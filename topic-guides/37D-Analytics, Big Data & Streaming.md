# Section 37D: Analytics, Big Data & Streaming

## The idea

Match the requirement to the unique service keyword.

| Requirement / keyword                              | Service                                             |
| -------------------------------------------------- | --------------------------------------------------- |
| **Spark / Hadoop**                                 | **Amazon EMR (Elastic MapReduce)**                  |
| **SQL directly on S3**                             | **Amazon Athena**                                   |
| **Query S3 data from Redshift without loading it** | **Amazon Redshift Spectrum**                        |
| **BI + standard SQL + analytical workloads**       | **Amazon Redshift**                                 |
| **Apache Flink / real-time stream processing**     | **Amazon Managed Service for Apache Flink**         |
| **Interactive Flink streaming analysis**           | **Flink Studio**                                    |
| **Kafka**                                          | **Amazon MSK (Managed Streaming for Apache Kafka)** |
| **RabbitMQ**                                       | **Amazon MQ** |
| **BI dashboards**                                  | **Amazon QuickSight**                               |
| **File-based video transcoding**                   | **AWS Elemental MediaConvert**                      |
| **Legacy video transcoding**                       | **Amazon Elastic Transcoder**                       |

---

# Amazon EMR (Elastic MapReduce)

**Amazon EMR = managed big-data processing using frameworks such as Apache Spark and Hadoop.**

Use it for:

* large-scale data processing
* Spark jobs
* Hadoop workloads

### Signal

> **Spark / Hadoop → Amazon EMR**

### Example

Large data in S3 must be processed with Apache Spark:

```text
S3
 ↓
Amazon EMR
 ↓
Spark processing
 ↓
Processed data
```

→ **Amazon EMR**

### Spot Instances with EMR

You can use **Spot Instances for suitable EMR task nodes** to reduce cost when the workload can tolerate interruption and the task nodes are suitable for Spot capacity.

### Memory

> **Large-scale Spark/Hadoop processing → Amazon EMR**

---

# Amazon EMR vs Amazon Redshift

| Amazon EMR                         | Amazon Redshift            |
| ---------------------------------- | -------------------------- |
| Managed big-data processing        | Managed data warehouse     |
| Spark / Hadoop                     | Analytical SQL             |
| Process / transform large datasets | BI / reporting / analytics |

### Signal

> **Spark / Hadoop → Amazon EMR**

> **BI + standard SQL + analytical workloads → Amazon Redshift**

A question can require **both**.

---

# EMR + Redshift Pattern

```text
S3 data lake
     ↓
Amazon EMR
(Spark / Hadoop)
     ↓
Amazon Redshift
     ↓
BI tools + standard SQL
```

```text
EMR
→ processes the large dataset

Redshift
→ stores/analyzes processed data
  for analytical SQL and BI
```

### Example

> "A company stores large datasets in S3 and wants to use big-data processing frameworks to process the data. Business users then need high-performance access using BI tools and standard SQL queries."

→ **Amazon EMR + Amazon Redshift**

---

# Amazon Athena

**Amazon Athena = serverless interactive SQL queries directly against data in S3.**

Use it when CSV, JSON, Parquet, or similar data is already in S3.

```text
S3
 ↓
Amazon Athena
 ↓
SQL query
```

No database server needs to be provisioned just to query the data.

### Signal

> **SQL + S3 → Amazon Athena**

### Memory

> **Amazon Athena = SQL ON S3**

---

## Athena + QuickSight Reporting

**Pattern: S3 data + weekly reporting + visualization + cost-effective → Glue Crawler + Athena + QuickSight**

```text
S3
 ↓
Glue Crawler
 ↓
Glue Data Catalog
 ↓
Athena
 ↓
QuickSight
```

| Service          | Role                          |
| ---------------- | ----------------------------- |
| **Glue Crawler** | Discovers the S3 data schema  |
| **Athena**       | Serverless SQL directly on S3 |
| **QuickSight**   | Dashboards / visualization    |

> A company is preparing a solution that the sales team can use for generating weekly revenue reports. The team must be able to run analysis on sales records stored in Amazon S3 and visualize the results of queries.
>
> How can the solutions architect meet the requirement in the most cost-effective way possible?

→ **Use AWS Glue crawler to build tables in AWS Glue Data Catalog. Run queries using Amazon Athena. Use Amazon QuickSight for visualization.**

### Exam signal

> **S3 + SQL + occasional/weekly reporting + visualization → Glue + Athena + QuickSight**

---

# Amazon Redshift

**Amazon Redshift = managed cloud data warehouse for analytical workloads.**

Use it for:

* analytical SQL queries
* high-performance analytics
* BI / reporting
* data warehousing

### Signal

> **BI + standard SQL + analytical workloads → Amazon Redshift**

## Redshift Cross-Region Snapshots

**Redshift Cross-Region Snapshot Copy** automatically copies snapshots of a **Redshift cluster** to another AWS Region.

Use it for **disaster recovery from an entire Region outage**.

```text
Redshift Cluster (Region A)
        ↓
Cross-Region Snapshot Copy
        ↓
Snapshot (Region B)
        ↓
Restore Redshift Cluster
```

### Signal

> **Redshift cluster + Region outage → Cross-Region Snapshot Copy**

**Automated snapshots alone are not enough for Regional DR** because the snapshot needs to be available in another Region.

---

# Amazon Redshift Spectrum

**Amazon Redshift Spectrum = query data stored in S3 directly from Redshift without loading the data into Redshift tables.**

Use it when:

* you already use **Amazon Redshift**
* some data remains in **S3**
* you want to query that S3 data using Redshift

```text
Amazon Redshift
      ↓
Redshift Spectrum
      ↓
S3 data
```

### Signal

> **Query S3 data from Redshift without loading it → Redshift Spectrum**

### Key distinction

```text
Athena
→ SQL directly on S3

Redshift Spectrum
→ SQL on S3 from Redshift
```

---

# Athena vs Redshift vs Redshift Spectrum

| Requirement                                           | Service               |
| ----------------------------------------------------- | --------------------- |
| Query data directly in S3 with SQL                    | **Amazon Athena**     |
| Query S3 data using an existing Redshift environment  | **Redshift Spectrum** |
| Dedicated analytical data warehouse                   | **Amazon Redshift**   |
| BI / reporting / high-performance warehouse analytics | **Amazon Redshift**   |

### Memory

```text
Athena
= QUERY S3

Redshift
= ANALYTICAL DATA WAREHOUSE

Redshift Spectrum
= QUERY S3 FROM REDSHIFT
```

### Examples

> "Analysts need to query files already stored in S3 using SQL, with no need to provision a database."

→ **Amazon Athena**

> "Business users need high-performance analytical SQL queries against a data warehouse."

→ **Amazon Redshift**

> "The company already uses Redshift but wants to query additional datasets stored in S3 without loading them into Redshift."

→ **Redshift Spectrum**

---

# Amazon Managed Service for Apache Flink

**Amazon Managed Service for Apache Flink = managed real-time stream processing with Apache Flink.**

Use it for:

* real-time analytics
* streaming ETL
* continuous event processing
* stream transformations
* detecting patterns in streams

Typical sources:

* Amazon Kinesis Data Streams
* Amazon MSK / Apache Kafka

Typical pattern:

```text
Kinesis / Kafka
      ↓
Amazon Managed Service for Apache Flink
      ↓
Real-time processing
      ↓
Kinesis / S3 / other destinations
```

### Signal

> **Apache Flink + real-time streaming analytics → Amazon Managed Service for Apache Flink**

### Example

> "A company receives continuous streaming data from Amazon Kinesis and needs to perform real-time analytics using Apache Flink."

→ **Amazon Managed Service for Apache Flink**

### Flink use cases

```text
Continuous event stream
        ↓
Flink
        ↓
Transform / aggregate / analyze
        ↓
Real-time results
```

Examples:

* real-time analytics
* streaming ETL
* continuous event processing
* stream transformations
* detecting patterns in streams

---

# Flink Studio

**Flink Studio = interactive environment for developing and analyzing Apache Flink streaming workloads.**

Use it for **interactive querying, exploration, development, or streaming analysis**.

### Signal

> **Interactive Flink development / streaming analysis → Flink Studio**

### Example

> "A company wants to interactively query and analyze streaming data using Apache Flink."

→ **Flink Studio**

---

# Flink Studio vs Managed Service for Apache Flink

| Requirement                                                 | Answer                                      |
| ----------------------------------------------------------- | ------------------------------------------- |
| Run managed production Flink stream-processing applications | **Amazon Managed Service for Apache Flink** |
| Interactive Flink development                               | **Flink Studio**                            |
| Interactive streaming analysis                              | **Flink Studio**                            |

### Memory

```text
Production stream processing
→ Amazon Managed Service for Apache Flink

Interactive development / exploration / analysis
→ Flink Studio
```

---

# Amazon MSK (Managed Streaming for Apache Kafka)

**Amazon MSK = managed Apache Kafka.**

Use it for Kafka-compatible workloads without managing the underlying Kafka infrastructure yourself.

### Signal

> **Kafka → Amazon MSK**

### Example

> "An existing application uses Apache Kafka and the company wants a managed Kafka service on AWS."

→ **Amazon MSK**

### Memory

> **MSK = MANAGED KAFKA**

---

# MSK vs Managed Service for Apache Flink

```text
Kafka
→ Amazon MSK

Process streaming data with Apache Flink
→ Amazon Managed Service for Apache Flink
```

Typical architecture:

```text
Applications
    ↓
Amazon MSK
(Kafka)
    ↓
Amazon Managed Service for Apache Flink
    ↓
Real-time processing
    ↓
S3 / Kinesis / other destination
```

### Distinction

| Service                                     | Role                       |
| ------------------------------------------- | -------------------------- |
| **Amazon MSK**                              | Streaming platform / Kafka |
| **Amazon Managed Service for Apache Flink** | Stream processing          |

---

# Amazon QuickSight

**Amazon QuickSight = business intelligence and dashboards.**

Use it for:

* dashboards
* charts
* reports
* business visualizations

### Signal

> **Business dashboard → Amazon QuickSight**

### Example

> "Business users need dashboards and visual reports based on analytical data."

→ **Amazon QuickSight**

### Analytics architecture

```text
Data
 ↓
Processing / warehouse
 ↓
Amazon Redshift
 ↓
Amazon QuickSight
 ↓
Dashboards / reports
```

QuickSight is the **visualization / BI layer**, not the primary big-data processing engine.

---

# AWS Elemental MediaConvert

**AWS Elemental MediaConvert = managed file-based video transcoding service.**

Use it to convert/process **video files for on-demand delivery**.

### Signal

> **File-based video transcoding → AWS Elemental MediaConvert**

### Example

```text
Video file
    ↓
MediaConvert
    ↓
Transcoded video
```

Example: create multiple resolutions or formats for on-demand playback.

---

# MediaConvert vs MediaLive

| Requirement                  | Service                        |
| ---------------------------- | ------------------------------ |
| File-based video transcoding | **AWS Elemental MediaConvert** |
| Live video encoding          | **AWS Elemental MediaLive**    |

### Memory

> **MediaConvert = FILE-BASED VIDEO**

> **MediaLive = LIVE VIDEO**

---

# Legacy Service: Amazon Elastic Transcoder

**Amazon Elastic Transcoder = older managed video/audio transcoding service.**

It was **discontinued on November 13, 2025**. AWS recommends **AWS Elemental MediaConvert** for file-based transcoding workflows.

### Exam memory

> **Video transcoding → MediaConvert**

> **Old question mentioning Elastic Transcoder → recognize it as the legacy service**

```text
Current file-based video transcoding
→ MediaConvert

Legacy service
→ Elastic Transcoder
```

Do not select Elastic Transcoder for a new modern AWS architecture.

---

# Amazon Timestream

**Amazon Timestream = fully managed time-series database for data that changes over time.**

Typical use cases:

* IoT sensor data
* application / infrastructure metrics
* monitoring data
* device telemetry

Example:

```text
10:00 → CPU = 45%
10:01 → CPU = 52%
10:02 → CPU = 61%
```

### Signal

> **Timestream = time-series data over time**

---

# Analytics Family

| Job                             | Service                                     |
| ------------------------------- | ------------------------------------------- |
| Big-data processing             | **Amazon EMR**                              |
| SQL on S3                       | **Amazon Athena**                           |
| Query S3 from Redshift          | **Amazon Redshift Spectrum**                |
| Data warehouse / analytical SQL | **Amazon Redshift**                         |
| Real-time Flink processing      | **Amazon Managed Service for Apache Flink** |
| Kafka                           | **Amazon MSK**                              |
| BI dashboards                   | **Amazon QuickSight**                       |
| Time-series data                | **Amazon Timestream**                       |

Streaming pattern:

```text
Kafka / Kinesis
      ↓
Apache Flink
      ↓
Real-time processing
```

Data warehouse pattern:

```text
Processed data
      ↓
Amazon Redshift
      ↓
BI / SQL analytics
```

---

# Service Comparison

| Service                                             | What it does                               | Signal keyword                     |
| --------------------------------------------------- | ------------------------------------------ | ---------------------------------- |
| **Amazon EMR (Elastic MapReduce)**                  | Managed big-data processing                | Spark / Hadoop                     |
| **Amazon Athena**                                   | SQL directly on S3                         | SQL on S3                          |
| **Amazon Redshift Spectrum**                        | Query S3 data from Redshift                | S3 + existing Redshift             |
| **Amazon Redshift**                                 | Data warehouse / analytical SQL            | BI / analytical workloads          |
| **Amazon Managed Service for Apache Flink**         | Managed real-time stream processing        | Apache Flink / real-time streaming |
| **Flink Studio**                                    | Interactive Flink development and analysis | Interactive Flink                  |
| **Amazon MSK (Managed Streaming for Apache Kafka)** | Managed Apache Kafka                       | Kafka                              |
| **Amazon QuickSight**                               | Business intelligence dashboards           | BI / dashboards                    |
| **Amazon Timestream**                               | Time-series database                       | IoT / metrics / telemetry          |
| **AWS Elemental MediaConvert**                      | File-based video transcoding               | Video transcoding                  |
| **Amazon Elastic Transcoder**                       | Legacy video/audio transcoding             | Old/legacy service                 |

---

# Important SAA Traps

## EMR vs Redshift

Do not choose Redshift just because the question says **large data**.

Look at the operation:

```text
Spark / Hadoop processing
→ Amazon EMR
```

```text
Analytical SQL / data warehouse / BI
→ Amazon Redshift
```

A question can require both:

```text
EMR
→ process

Redshift
→ analyze
```

---

## Athena vs Redshift

```text
SQL directly against S3
→ Amazon Athena
```

```text
Dedicated analytical warehouse
→ Amazon Redshift
```

---

## Athena vs Redshift Spectrum

```text
Query S3 directly
→ Amazon Athena
```

```text
Already using Redshift + need to query S3
→ Redshift Spectrum
```

**Spectrum does not mean you must load the S3 data into Redshift first.**

---

## MSK vs Flink

```text
Kafka
→ Amazon MSK
```

```text
Process streams using Apache Flink
→ Amazon Managed Service for Apache Flink
```

They can work together.

---

## Flink Studio vs Managed Service for Apache Flink

```text
Production stream-processing application
→ Amazon Managed Service for Apache Flink

Interactive Flink exploration / analysis
→ Flink Studio
```

---

## MediaConvert vs MediaLive

```text
Video files
→ MediaConvert
```

```text
Live video stream
→ MediaLive
```

---

## Redshift Regional DR

Redshift automated snapshots
→ ENABLED by default
→ default retention: 1 day

Cross-Region Snapshot Copy
→ NOT enabled by default
→ must configure destination Region

---

# Common Question Patterns

> **"Managed Apache Spark processing."**

→ **Amazon EMR**

> **"A company needs to process a large dataset using Apache Hadoop."**

→ **Amazon EMR**

> **"Query files in S3 using SQL."**

→ **Amazon Athena**

> **"Business users need high-performance analytical SQL queries and BI access."**

→ **Amazon Redshift**

> **"The company already has Redshift and needs to query files in S3 without loading them into Redshift."**

→ **Amazon Redshift Spectrum**

> **"A company stores large datasets in S3 and wants big-data processing frameworks to process the data. Business users then need high-performance access using BI tools and standard SQL queries."**

→ **Amazon EMR + Amazon Redshift**

```text
EMR
→ process

Redshift
→ analyze
```

> **"A company receives continuous streaming data from Amazon Kinesis and needs to perform real-time analytics using Apache Flink."**

→ **Amazon Managed Service for Apache Flink**

> **"A company wants to interactively query and analyze streaming data using Apache Flink."**

→ **Flink Studio**

> **"An existing workload uses Apache Kafka."**

→ **Amazon MSK**

> **"Business users need dashboards and visual reports."**

→ **Amazon QuickSight**

> **"File-based video transcoding for on-demand content."**

→ **AWS Elemental MediaConvert**

> **"An old question mentions Elastic Transcoder."**

→ **Recognize it as the legacy service**

> **"A Redshift cluster must remain recoverable if an entire AWS Region becomes unavailable."**

→ **Enable Cross-Region Snapshot Copy**

> **"A company is preparing a solution that the sales team can use for generating weekly revenue reports. The team must be able to run analysis on sales records stored in Amazon S3 and visualize the results of queries."**

→ **Use AWS Glue crawler to build tables in AWS Glue Data Catalog. Run queries using Amazon Athena. Use Amazon QuickSight for visualization.**

---

# Pocket Card

| Keyword                                  | Answer                                              |
| ---------------------------------------- | --------------------------------------------------- |
| Spark / Hadoop                           | **Amazon EMR (Elastic MapReduce)**                  |
| Large-scale big-data processing          | **Amazon EMR**                                      |
| SQL on S3                                | **Amazon Athena**                                   |
| S3 + existing Redshift                   | **Redshift Spectrum**                               |
| Data warehouse                           | **Amazon Redshift**                                 |
| BI + standard SQL + analytical workloads | **Amazon Redshift**                                 |
| **Redshift + Region outage**             | **Cross-Region Snapshot Copy**                      |
| Apache Flink / real-time streaming       | **Amazon Managed Service for Apache Flink**         |
| Interactive Flink streaming analysis     | **Flink Studio**                                    |
| Kafka                                    | **Amazon MSK (Managed Streaming for Apache Kafka)** |
| BI dashboards                            | **Amazon QuickSight**                               |
| Time-series data / IoT / metrics         | **Amazon Timestream**                               |
| File-based video transcoding             | **AWS Elemental MediaConvert**                      |
| Live video encoding                      | **AWS Elemental MediaLive**                         |
| Legacy video transcoding                 | **Amazon Elastic Transcoder — discontinued**        |
| S3 + weekly reports + visualization      | **Glue + Athena + QuickSight**                      |
