# Section 37D: Analytics, Big Data & Streaming

## The idea

These are AWS services that commonly appear in SAA questions involving **big-data processing, analytical SQL, streaming data, Kafka, Apache Flink, BI dashboards, and video processing**.

You generally don't need deep knowledge of each one.

The best strategy is:

> **Read the requirement → identify the unique keyword → choose the service.**

For example:

```text
Spark / Hadoop
→ Amazon EMR

SQL directly on S3
→ Amazon Athena

BI + standard SQL + analytical workloads
→ Amazon Redshift

Apache Flink / real-time stream processing
→ Managed Service for Apache Flink

Interactive Flink streaming analysis
→ Flink Studio

Kafka
→ Amazon MSK

BI dashboards
→ Amazon QuickSight

File-based video transcoding
→ AWS Elemental MediaConvert

Legacy video transcoding service
→ Amazon Elastic Transcoder
```

---

# Amazon EMR

**Amazon EMR = managed big-data processing using frameworks such as Apache Spark and Hadoop.**

Use it for:

* large-scale data processing
* Spark jobs
* Hadoop workloads

### Signal

> **Spark / Hadoop → EMR**

---

## Example

A company has a large amount of data stored in S3 and wants to process it using Apache Spark.

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

---

## Spot Instances with EMR

You can use **Spot Instances for suitable EMR task nodes** to reduce cost.

This is useful when the workload can tolerate interruption and the task nodes are suitable for Spot capacity.

### Memory

> **Large-scale Spark/Hadoop processing → EMR**

---

# Amazon EMR vs Amazon Redshift

These services can both appear in the same architecture, but they solve different problems.

```text
EMR
= managed big-data processing
= Spark / Hadoop
= process / transform large datasets

Redshift
= managed data warehouse
= analytical SQL queries
= BI / reporting / analytics
```

### Important distinction

> **Spark / Hadoop → EMR**

> **BI + standard SQL + analytical workloads → Redshift**

A question can require **both** services.

---

# EMR + Redshift Pattern

A common architecture is:

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

Here:

```text
EMR
→ processes the large dataset

Redshift
→ stores/analyzes the processed data
  for analytical SQL and BI workloads
```

---

## Example

> "A company stores large datasets in S3 and wants to use big-data processing frameworks to process the data. Business users then need high-performance access using BI tools and standard SQL queries."

→ **Amazon EMR + Amazon Redshift**

### Signal

```text
Spark / Hadoop
→ EMR

BI + standard SQL + analytical workloads
→ Redshift
```

---

# Amazon Athena

**Amazon Athena = query data in S3 using SQL.**

Athena is a serverless interactive query service.

### Signal

> **SQL + S3 → Athena**

---

## Example

Suppose CSV, JSON, or Parquet data is already stored in S3.

```text
S3
 ↓
Athena
 ↓
SQL query
```

No database server needs to be provisioned just to query the data.

---

## Important distinction

```text
Athena
= query data directly in S3

Redshift
= data warehouse for analytical workloads
```

### Simple memory

> **Athena = SQL ON S3**

---

# Amazon Redshift

**Amazon Redshift = managed cloud data warehouse designed for analytical workloads.**

Use it when users need:

* analytical SQL queries
* high-performance analytics
* BI/reporting
* data warehousing

### Signal

> **BI + standard SQL + analytical workloads → Redshift**

---

## Athena vs Redshift

This distinction is especially useful in SAA questions.

```text
Data is in S3
+
Need to query it directly with SQL
→ Athena
```

```text
Need a dedicated analytical data warehouse
+
BI / reporting / high-performance SQL
→ Redshift
```

### Mental model

```text
Athena
= QUERY S3

Redshift
= ANALYTICAL DATA WAREHOUSE
```

---

# Managed Service for Apache Flink

**Amazon Managed Service for Apache Flink = managed real-time stream processing using Apache Flink.**

Use it when you need to process **streaming data continuously and in real time**.

It is useful for:

* real-time analytics
* streaming ETL
* processing events continuously
* transforming streaming data
* detecting patterns in streams

Typical streaming sources include:

* Amazon Kinesis Data Streams
* Amazon MSK / Apache Kafka

Typical pattern:

```text
Kinesis / Kafka
      ↓
Managed Service for Apache Flink
      ↓
Real-time processing
      ↓
Kinesis / S3 / other destinations
```

### Signal

> **Apache Flink + real-time streaming analytics → Managed Service for Apache Flink**

---

## Example

> "A company receives continuous streaming data from Amazon Kinesis and needs to perform real-time analytics using Apache Flink."

→ **Amazon Managed Service for Apache Flink**

---

## Flink use cases

Managed Service for Apache Flink can be used for workloads such as:

```text
Continuous event stream
        ↓
Flink
        ↓
Transform / aggregate / analyze
        ↓
Real-time results
```

Examples include:

* real-time analytics
* streaming ETL
* continuous event processing
* stream transformations
* detecting patterns in streams

---

# Flink Studio

**Flink Studio = interactive environment for developing and analyzing Apache Flink streaming workloads.**

Use it when you need to **interactively query, explore, or develop Flink streaming applications**.

### Signal

> **Interactive Flink development / streaming analysis → Flink Studio**

---

## Example

> "A company wants to interactively query and analyze streaming data using Apache Flink."

→ **Flink Studio**

---

# Flink Studio vs Managed Service for Apache Flink

These names are very similar, so focus on the requirement.

```text
Managed Service for Apache Flink
= run managed Flink stream-processing applications

Flink Studio
= interactively develop / analyze Flink streaming workloads
```

### Mental model

```text
Need a managed production Flink application
→ Managed Service for Apache Flink

Need interactive development / exploration / analysis
→ Flink Studio
```

### Signal comparison

| Requirement                                    | Answer                               |
| ---------------------------------------------- | ------------------------------------ |
| Apache Flink + real-time streaming application | **Managed Service for Apache Flink** |
| Interactive Flink development                  | **Flink Studio**                     |
| Interactive streaming analysis                 | **Flink Studio**                     |

---

# Amazon MSK

**Amazon Managed Streaming for Apache Kafka (Amazon MSK) = managed Apache Kafka.**

You can run Kafka-compatible workloads without managing the underlying Kafka infrastructure yourself.

### Signal

> **Kafka → MSK**

Don't overthink it.

---

## Example

> "An existing application uses Apache Kafka and the company wants a managed Kafka service on AWS."

→ **Amazon MSK**

### Memory

> **MSK = MANAGED KAFKA**

---

# MSK vs Managed Service for Apache Flink

These services may appear together because Kafka can provide the streaming data and Flink can process it.

```text
Kafka
→ MSK

Process streaming data with Apache Flink
→ Managed Service for Apache Flink
```

Typical architecture:

```text
Applications
    ↓
Amazon MSK
(Kafka)
    ↓
Managed Service for Apache Flink
    ↓
Real-time processing
    ↓
S3 / Kinesis / other destination
```

### Important distinction

```text
MSK
= STREAMING PLATFORM / KAFKA

Flink
= STREAM PROCESSING
```

---

# Amazon QuickSight

**Amazon QuickSight = business intelligence and dashboards.**

Use it to create:

* dashboards
* charts
* reports
* business visualizations

### Signal

> **Business dashboard → QuickSight**

---

## Example

> "Business users need dashboards and visual reports based on analytical data."

→ **Amazon QuickSight**

---

## QuickSight in an analytics architecture

A common architecture can look like:

```text
Data
 ↓
Processing / warehouse
 ↓
Amazon Redshift
 ↓
QuickSight
 ↓
Dashboards / reports
```

QuickSight is the **visualization / BI layer**, not the primary big-data processing engine.

---

# AWS Elemental MediaConvert

**AWS Elemental MediaConvert = managed file-based video transcoding service.**

Use it to convert and process **video files for on-demand delivery**.

### Signal

> **File-based video transcoding → MediaConvert**

---

## Example

```text
Video file
    ↓
MediaConvert
    ↓
Transcoded video
```

For example, a company may upload a video and need versions in different resolutions or formats for on-demand playback.

→ **AWS Elemental MediaConvert**

---

# MediaConvert vs MediaLive

A common distinction is:

```text
File-based video transcoding
→ MediaConvert

Live video encoding
→ MediaLive
```

### Memory

> **MediaConvert = FILE-BASED VIDEO**

> **MediaLive = LIVE VIDEO**

---

# Legacy Service: Amazon Elastic Transcoder

**Amazon Elastic Transcoder was the older managed video/audio transcoding service.**

It was **discontinued on November 13, 2025**.

AWS recommends **MediaConvert** for file-based transcoding workflows.

### SAA memory

> **Video transcoding → MediaConvert**

> **Old question mentioning Elastic Transcoder → recognize it as the legacy service**

---

## Important distinction

```text
Current file-based video transcoding
→ MediaConvert

Legacy service
→ Elastic Transcoder
```

Do not select Elastic Transcoder for a new modern AWS architecture.

---

# Analytics Family

A useful way to group these services is by the job they perform.

```text
             ANALYTICS
                 │
      ┌──────────┼──────────┐
      ↓          ↓          ↓
   Process      Query      Visualize
      │          │          │
     EMR       Athena    QuickSight
      │
 Spark / Hadoop
```

For streaming:

```text
Kafka / Kinesis
      ↓
Flink
      ↓
Real-time processing
```

For data warehousing:

```text
Processed data
      ↓
Redshift
      ↓
BI / SQL analytics
```

---

# Service Comparison

| Service                              | What it does                               | Signal keyword                     |
| ------------------------------------ | ------------------------------------------ | ---------------------------------- |
| **Amazon EMR**                       | Managed big-data processing                | Spark / Hadoop                     |
| **Amazon Athena**                    | SQL directly on S3                         | SQL on S3                          |
| **Amazon Redshift**                  | Data warehouse / analytical SQL            | BI / analytical workloads          |
| **Managed Service for Apache Flink** | Managed real-time stream processing        | Apache Flink / real-time streaming |
| **Flink Studio**                     | Interactive Flink development and analysis | Interactive Flink                  |
| **Amazon MSK**                       | Managed Apache Kafka                       | Kafka                              |
| **Amazon QuickSight**                | Business intelligence dashboards           | BI / dashboards                    |
| **MediaConvert**                     | File-based video transcoding               | Video transcoding                  |
| **Elastic Transcoder**               | Legacy video/audio transcoding             | Old/legacy service                 |

---

# Important SAA Traps

## EMR vs Redshift

Don't choose Redshift just because the question says "large data."

Look for the actual operation:

```text
Spark / Hadoop processing
→ EMR
```

```text
Analytical SQL / data warehouse / BI
→ Redshift
```

A question can require both:

```text
EMR
→ process

Redshift
→ analyze
```

---
### Amazon Timestream

**Amazon Timestream** is a fully managed **time-series database** designed for data that changes over time.

Typical use cases:

* IoT sensor data
* Application and infrastructure metrics
* Monitoring data
* Device telemetry

Example:

```text
10:00 → CPU = 45%
10:01 → CPU = 52%
10:02 → CPU = 61%
```

**Remember:**

> **Timestream = time-series data over time**
 
## Athena vs Redshift

The key distinction is where and how the data is queried.

```text
SQL directly against S3
→ Athena
```

```text
Dedicated analytical warehouse
→ Redshift
```

### Example

> "Analysts need to query files already stored in S3 using SQL, with no need to provision a database."

→ **Athena**

---

> "Business users need high-performance analytical SQL queries against a data warehouse."

→ **Redshift**

---

## MSK vs Flink

Don't confuse the streaming platform with the stream processor.

```text
Kafka
→ MSK
```

```text
Process streams using Apache Flink
→ Managed Service for Apache Flink
```

They can work together.

---

## Flink Studio vs Managed Service for Apache Flink

Look for **interactive development/analysis** versus a managed streaming application.

```text
Production stream-processing application
→ Managed Service for Apache Flink

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
          │      EMR
          │
          ├── SQL directly on S3?
          │       ↓
          │     Athena
          │
          ├── Data warehouse / BI / analytical SQL?
          │       ↓
          │    Redshift
          │
          ├── Kafka?
          │       ↓
          │      MSK
          │
          ├── Apache Flink + real-time processing?
          │       ↓
          │   Managed Service
          │   for Apache Flink
          │
          ├── Interactive Flink analysis?
          │       ↓
          │   Flink Studio
          │
          ├── BI dashboards?
          │       ↓
          │   QuickSight
          │
          └── File-based video transcoding?
                  ↓
             MediaConvert
```

---

# Common Question Patterns

> **"Managed Apache Spark processing."**

→ **Amazon EMR**

---

> **"A company needs to process a large dataset using Apache Hadoop."**

→ **Amazon EMR**

---

> **"Query files in S3 using SQL."**

→ **Amazon Athena**

---

> **"Business users need high-performance analytical SQL queries and BI access."**

→ **Amazon Redshift**

---

> **"A company stores large datasets in S3 and wants big-data processing frameworks to process the data. Business users then need high-performance access using BI tools and standard SQL queries."**

→ **Amazon EMR + Amazon Redshift**

```text
EMR
→ process

Redshift
→ analyze
```

---

> **"A company receives continuous streaming data from Amazon Kinesis and needs to perform real-time analytics using Apache Flink."**

→ **Amazon Managed Service for Apache Flink**

---

> **"A company wants to interactively query and analyze streaming data using Apache Flink."**

→ **Flink Studio**

---

> **"An existing workload uses Apache Kafka."**

→ **Amazon MSK**

---

> **"Business users need dashboards and visual reports."**

→ **Amazon QuickSight**

---

> **"File-based video transcoding for on-demand content."**

→ **AWS Elemental MediaConvert**

---

> **"An old question mentions Elastic Transcoder."**

→ **Recognize it as the legacy service**

---

# Pocket Card

| Keyword                                  | Answer                                |
| ---------------------------------------- | ------------------------------------- |
| Spark / Hadoop                           | **Amazon EMR**                        |
| Large-scale big-data processing          | **Amazon EMR**                        |
| SQL on S3                                | **Amazon Athena**                     |
| Data warehouse                           | **Amazon Redshift**                   |
| BI + standard SQL + analytical workloads | **Amazon Redshift**                   |
| Apache Flink / real-time streaming       | **Managed Service for Apache Flink**  |
| Interactive Flink streaming analysis     | **Flink Studio**                      |
| Kafka                                    | **Amazon MSK**                        |
| BI dashboards                            | **Amazon QuickSight**                 |
| File-based video transcoding             | **MediaConvert**                      |
| Live video encoding                      | **MediaLive**                         |
| Legacy video transcoding                 | **Elastic Transcoder — discontinued** |

---

# Final Memory

```text
EMR
= SPARK / HADOOP
= BIG-DATA PROCESSING

Athena
= SQL ON S3

Redshift
= ANALYTICAL DATA WAREHOUSE
= BI + STANDARD SQL

Managed Service for Apache Flink
= REAL-TIME STREAM PROCESSING
= APACHE FLINK

Flink Studio
= INTERACTIVE FLINK STREAM ANALYSIS

MSK
= KAFKA

QuickSight
= BI DASHBOARDS

MediaConvert
= FILE-BASED VIDEO TRANSCODING

Elastic Transcoder
= LEGACY / DISCONTINUED
```

# The Golden Rule

```text
Spark
→ EMR

Hadoop
→ EMR

SQL on S3
→ Athena

BI + analytical SQL
→ Redshift

Kafka
→ MSK

Apache Flink + real-time streaming
→ Managed Service for Apache Flink

Interactive Flink analysis
→ Flink Studio

BI dashboards
→ QuickSight

File-based video transcoding
→ MediaConvert
```

> **Don't memorize the implementation.**
>
> **Memorize the unique signal.**

For example:

```text
Spark / Hadoop       → EMR
SQL on S3            → Athena
Data warehouse       → Redshift
Kafka                → MSK
Apache Flink         → Managed Service for Apache Flink
Interactive Flink   → Flink Studio
BI dashboard         → QuickSight
Video transcoding    → MediaConvert
```
