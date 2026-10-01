# Section 37D: Analytics, Big Data & Streaming

## The idea

Match the requirement to the unique service keyword.

| Requirement / keyword                          | Service                                             |
| ---------------------------------------------- | --------------------------------------------------- |
| **Spark / Hadoop**                             | **Amazon EMR (Elastic MapReduce)**                  |
| **SQL directly on S3**                         | **Amazon Athena**                                   |
| **BI + standard SQL + analytical workloads**   | **Amazon Redshift**                                 |
| **Apache Flink / real-time stream processing** | **Amazon Managed Service for Apache Flink**         |
| **Interactive Flink streaming analysis**       | **Flink Studio**                                    |
| **Kafka**                                      | **Amazon MSK (Managed Streaming for Apache Kafka)** |
| **BI dashboards**                              | **Amazon QuickSight**                               |
| **File-based video transcoding**               | **AWS Elemental MediaConvert**                      |
| **Legacy video transcoding**                   | **Amazon Elastic Transcoder**                       |

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

# Amazon Redshift

**Amazon Redshift = managed cloud data warehouse for analytical workloads.**

Use it for:

* analytical SQL queries
* high-performance analytics
* BI / reporting
* data warehousing

### Signal

> **BI + standard SQL + analytical workloads → Amazon Redshift**

---

# Athena vs Redshift

| Requirement                           | Service             |
| ------------------------------------- | ------------------- |
| Query data directly in S3 with SQL    | **Amazon Athena**   |
| Dedicated analytical data warehouse   | **Amazon Redshift** |
| BI / reporting / high-performance SQL | **Amazon Redshift** |

### Mental model

```text
Athena
= QUERY S3

Redshift
= ANALYTICAL DATA WAREHOUSE
```

### Examples

> "Analysts need to query files already stored in S3 using SQL, with no need to provision a database."

→ **Amazon Athena**

> "Business users need high-performance analytical SQL queries against a data warehouse."

→ **Amazon Redshift**

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

# Analytics Decision Tree

```text
What is the requirement?
          │
          ├── Spark / Hadoop?
          │       ↓
          │   Amazon EMR
          │
          ├── SQL directly on S3?
          │       ↓
          │   Amazon Athena
          │
          ├── Data warehouse / BI / analytical SQL?
          │       ↓
          │   Amazon Redshift
          │
          ├── Kafka?
          │       ↓
          │   Amazon MSK
          │
          ├── Apache Flink + real-time processing?
          │       ↓
          │   Amazon Managed Service
          │   for Apache Flink
          │
          ├── Interactive Flink analysis?
          │       ↓
          │   Flink Studio
          │
          ├── BI dashboards?
          │       ↓
          │   Amazon QuickSight
          │
          └── File-based video transcoding?
                  ↓
            AWS Elemental MediaConvert
```

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

---

# Pocket Card

| Keyword                                  | Answer                                              |
| ---------------------------------------- | --------------------------------------------------- |
| Spark / Hadoop                           | **Amazon EMR (Elastic MapReduce)**                  |
| Large-scale big-data processing          | **Amazon EMR**                                      |
| SQL on S3                                | **Amazon Athena**                                   |
| Data warehouse                           | **Amazon Redshift**                                 |
| BI + standard SQL + analytical workloads | **Amazon Redshift**                                 |
| Apache Flink / real-time streaming       | **Amazon Managed Service for Apache Flink**         |
| Interactive Flink streaming analysis     | **Flink Studio**                                    |
| Kafka                                    | **Amazon MSK (Managed Streaming for Apache Kafka)** |
| BI dashboards                            | **Amazon QuickSight**                               |
| Time-series data / IoT / metrics         | **Amazon Timestream**                               |
| File-based video transcoding             | **AWS Elemental MediaConvert**                      |
| Live video encoding                      | **AWS Elemental MediaLive**                         |
| Legacy video transcoding                 | **Amazon Elastic Transcoder — discontinued**        |

---

