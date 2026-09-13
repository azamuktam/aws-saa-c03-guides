# Section 37: The Gap-Fill Services

## The idea

These are smaller AWS services that usually appear in SAA questions as **specific use cases**.

You generally don't need deep knowledge of each one.

The best strategy is:

> **Read the requirement → identify the unique keyword → choose the service.**

For example:

```text
Existing RabbitMQ application
→ Amazon MQ

Lift-and-shift servers to AWS
→ AWS Application Migration Service (MGN)

Disaster recovery for servers
→ AWS Elastic Disaster Recovery (DRS)

TLS certificates
→ ACM

IPv6 outbound-only Internet access
→ Egress-Only Internet Gateway

Query S3 using SQL
→ Athena
```

---

# Migration & Messaging

## Amazon MQ

**Amazon MQ = managed traditional message broker.**

It is useful when an existing application already uses a traditional broker and you want to migrate it to AWS with **minimal application changes**.

Supports:

* ActiveMQ
* RabbitMQ

Common keywords:

```text
RabbitMQ
ActiveMQ
JMS
AMQP
Existing message broker
Minimal code changes
```

### Important distinction

```text
Existing broker-based application
→ Amazon MQ

New AWS-native application
→ SQS / SNS / EventBridge
```

### Example

> An existing application uses RabbitMQ and must migrate to AWS with minimal code changes.

→ **Amazon MQ**

### Memory

> **Existing RabbitMQ / ActiveMQ → Amazon MQ**

---

## AWS Application Migration Service (MGN)

**MGN = rehost / lift-and-shift servers to AWS.**

It continuously replicates the source server's **block-level data** to AWS and allows the server to be launched as an EC2 instance.

```text
On-premises server
        ↓
     MGN agent
        ↓
Continuous replication
        ↓
AWS staging area
        ↓
      EC2
```

Use it when:

> "Move existing physical or virtual servers to AWS with minimal changes."

### Memory

> **Rehost / lift-and-shift → MGN**

---

## AWS Elastic Disaster Recovery (DRS)

**DRS = disaster recovery for servers.**

It continuously replicates workloads into AWS so that recovery instances can be launched when the source environment fails.

```text
Production server
      ↓
Continuous replication
      ↓
AWS recovery environment

Disaster
   ↓
Launch recovery instances
```

Use it when:

> "We need a disaster recovery solution for physical, virtual, or cloud servers."

### MGN vs DRS

```text
MGN
= migrate to AWS

DRS
= recover in AWS during a disaster
```

Both use continuous replication, but the **purpose** is different.

---

## AWS DMS + SCT

### AWS Database Migration Service (DMS)

**DMS = migrate / replicate database data.**

Use it when moving data between databases.

### AWS Schema Conversion Tool (SCT)

**SCT = convert database schema and code when changing database engines.**

Example:

```text
Oracle
   ↓
SCT
   ↓
Converted PostgreSQL schema
   ↓
DMS
   ↓
Aurora PostgreSQL
```

### Memory

> **DMS = move data**

> **SCT = convert schema**

### Common pattern

```text
Same engine
→ DMS

Different engines
→ SCT + DMS
```

---

# The 7 Rs of Migration

The 7 Rs describe different migration strategies.

| R                           | Meaning                    | Simple idea               |
| --------------------------- | -------------------------- | ------------------------- |
| **Rehost**                  | Move without major changes | Lift and shift            |
| **Replatform**              | Move with limited changes  | Use a managed AWS service |
| **Repurchase**              | Replace the application    | Buy SaaS                  |
| **Refactor / Re-architect** | Redesign the application   | Build cloud-native        |
| **Relocate**                | Move the whole environment | VMware Cloud on AWS       |
| **Retain**                  | Keep it where it is        | Don't migrate yet         |
| **Retire**                  | Stop using it              | Decommission it           |

### Examples

```text
Rehost
On-prem VM → EC2

Replatform
MySQL on EC2 → Amazon RDS

Repurchase
Self-hosted CRM → SaaS CRM

Refactor
Monolith → Lambda / containers / serverless

Relocate
VMware environment → VMware Cloud on AWS

Retain
Keep workload on-premises

Retire
Delete unused application
```

---

# Certificates & Network Services

## ACM — AWS Certificate Manager

**ACM = TLS/SSL certificates for AWS services.**

Use it for HTTPS.

Common integrations:

* ALB
* CloudFront
* API Gateway

### Important facts

* ACM public certificates are provided at no additional charge.
* ACM can automatically renew eligible certificates.
* CloudFront certificates must be in **us-east-1**.
* Regional services such as ALB normally use a certificate in the same Region.

### Exam trap

You generally cannot export the private key of an **ACM public certificate** for installation on an EC2 server.

```text
HTTPS on ALB
→ ACM

HTTPS on CloudFront
→ ACM in us-east-1
```

### Memory

> **TLS/HTTPS certificate → ACM**

---

## Egress-Only Internet Gateway

**Egress-Only Internet Gateway = outbound-only Internet access for IPv6.**

```text
IPv6 EC2
   ↓
Egress-Only Internet Gateway
   ↓
Internet
```

Allows:

```text
EC2 → Internet ✅
Internet → EC2 ❌
```

For IPv4 private instances, the equivalent pattern is normally a **NAT Gateway**.

### Memory

> **IPv6 + outbound only → Egress-Only Internet Gateway**

---

## Route 53 Resolver Endpoints

These connect **AWS DNS resolution with on-premises DNS**.

### Inbound endpoint

Queries come **into AWS**:

```text
On-premises
    ↓
Inbound Resolver Endpoint
    ↓
AWS private DNS
```

Use it when:

> **On-premises systems need to resolve private DNS names in AWS.**

### Outbound endpoint

Queries go **out of AWS**:

```text
AWS VPC
   ↓
Outbound Resolver Endpoint
   ↓
On-premises DNS
```

Use it when:

> **AWS resources need to resolve internal/on-premises DNS names.**

### Memory

```text
Inbound
= query comes INTO AWS

Outbound
= query goes OUT OF AWS
```

---

# Edge & Hybrid Infrastructure

| Service         | What it does                                  | Signal                     |
| --------------- | --------------------------------------------- | -------------------------- |
| **Outposts**    | AWS infrastructure in your own data center    | AWS on-premises            |
| **Local Zones** | AWS infrastructure closer to a specific city  | Very low latency to a city |
| **Wavelength**  | AWS infrastructure inside telecom 5G networks | 5G / mobile edge           |

## Outposts

**AWS Outposts = AWS infrastructure physically installed in your data center.**

Use it when workloads need to remain on-premises while using AWS infrastructure/APIs.

```text
Your data center
       ↓
   AWS Outposts
       ↓
AWS infrastructure
```

### Signal

> **AWS infrastructure in your own data center → Outposts**

---

## Local Zones

**Local Zones = AWS infrastructure placed closer to users in a specific metropolitan area.**

Use it when an application needs extremely low latency for users in a particular city.

### Signal

> **Specific city + very low latency → Local Zones**

---

## Wavelength

**AWS Wavelength = AWS infrastructure inside telecom 5G networks.**

Use it for:

* mobile applications
* 5G workloads
* ultra-low-latency mobile applications

### Signal

> **5G → Wavelength**

---

# Analytics Family

| Service               | What it does                                 | Signal                     |
| --------------------- | -------------------------------------------- | -------------------------- |
| **AWS Glue**          | Serverless ETL + Data Catalog                | ETL / schema / catalog     |
| **AWS Glue Crawler**  | Discovers data and schema                    | Discover schema            |
| **Glue Data Catalog** | Stores metadata about datasets               | Metadata / tables / schema |
| **Glue ETL**          | Performs transformations                     | CSV → Parquet / ETL        |
| **Amazon EMR**        | Managed big-data processing                  | Spark / Hadoop             |
| **Amazon MSK**        | Managed Apache Kafka                         | Kafka                      |
| **Athena**            | SQL directly on S3                           | SQL on S3                  |
| **QuickSight**        | Business intelligence dashboards             | BI / dashboards            |
| **Lake Formation**    | Data lake governance and fine-grained access | Data lake permissions      |
| **AppFlow**           | Transfer data between SaaS and AWS           | Salesforce → S3            |

---

## AWS Glue

**AWS Glue = serverless data integration / ETL platform.**

Glue can:

* discover data
* catalog schemas
* transform data
* prepare data for analytics

### The 3 Glue pieces you should remember

```text
Glue Crawler
= discovers data and schema

Glue Data Catalog
= stores metadata

Glue ETL
= performs the transformation
```

Example:

```text
CSV files in S3
      ↓
Glue Crawler
      ↓
Discover schema
      ↓
Glue Data Catalog
      ↓
Store metadata/table definition
      ↓
Glue ETL job
      ↓
CSV → Parquet
      ↓
S3 transformed bucket
```

### Important distinction

**Crawler does NOT perform the transformation.**

It discovers the structure of the data and updates the Data Catalog.

**Data Catalog does NOT transform data.**

It stores metadata such as:

* table definitions
* columns
* data types
* locations

**Glue ETL performs the actual transformation.**

### Signal

> **Serverless ETL / Data Catalog / schema discovery → Glue**

> **Discover schema → Glue Crawler**

> **Store metadata → Glue Data Catalog**

> **Transform data → Glue ETL**

---

## Glue ETL + S3

For example:

```text
S3
 ↓
CSV
 ↓
Glue Crawler
 ↓
Data Catalog
 ↓
Glue ETL
 ↓
Parquet
 ↓
S3
```

Glue is a strong choice when you need **serverless ETL without managing servers or Spark clusters yourself**.

### Common trap

```text
Glue Crawler
= discovers schema

Glue ETL
= transforms data
```

Don't choose a crawler when the question asks you to **convert CSV to Parquet**.

---

## EMR

**Amazon EMR = managed big-data processing.**

Common frameworks:

* Apache Spark
* Hadoop

Use it for large-scale data processing when you specifically need those frameworks.

### Signal

> **Spark / Hadoop → EMR**

---

## MSK

**Amazon MSK = managed Apache Kafka.**

### Signal

> **Kafka → MSK**

Don't overthink it.

---

## Athena

**Amazon Athena = serverless SQL queries directly against S3.**

### Signal

> **SQL + S3 → Athena**

---

## QuickSight

**Amazon QuickSight = business intelligence and dashboards.**

Use it for:

* dashboards
* charts
* reports
* business visualization

### Signal

> **Business dashboards → QuickSight**

---

## Lake Formation

**AWS Lake Formation = build and govern a data lake with centralized, fine-grained access control.**

It can manage permissions at levels such as:

* tables
* columns
* rows

### Signal

> **Data lake + fine-grained permissions → Lake Formation**

---

## AppFlow

**Amazon AppFlow = transfer data between SaaS applications and AWS services.**

Example:

```text
Salesforce
    ↓
AppFlow
    ↓
S3
```

### Signal

> **SaaS → AWS data transfer → AppFlow**

---

# ML One-Liners

| Service         | What it does                                   | Signal                     |
| --------------- | ---------------------------------------------- | -------------------------- |
| **Rekognition** | Analyzes images and videos                     | Faces / objects / video    |
| **Transcribe**  | Speech → text                                  | Audio → transcript         |
| **Polly**       | Text → speech                                  | App reads text aloud       |
| **Translate**   | Text → another language                        | English → French           |
| **Comprehend**  | Understands text                               | Sentiment / entities       |
| **Textract**    | Extracts text, tables and forms from documents | Invoice / scanned form     |
| **Kendra**      | Searches enterprise documents                  | "Find our vacation policy" |
| **Personalize** | Personalized recommendations                   | Recommended products       |
| **Forecast**    | Time-series forecasting                        | Predict future demand      |
| **Lex**         | Conversational chatbot                         | Customer-service bot       |
| **SageMaker**   | Build/train/deploy custom ML models            | Train your own model       |

### Core distinctions

```text
Image / video
→ Rekognition

Speech → text
→ Transcribe

Text → speech
→ Polly

Translate text
→ Translate

Understand text / sentiment
→ Comprehend

Scanned document → structured information
→ Textract

Search company documents
→ Kendra

Recommendations
→ Personalize

Predict future values
→ Forecast

Chatbot
→ Lex

Build your own ML model
→ SageMaker
```

### Common trap

> "Extract names, fields, tables, and values from scanned invoices."

→ **Textract**

Not Rekognition.

```text
Rekognition
= image/video analysis

Textract
= document/text/table/form extraction
```

---

# Systems Manager (SSM) Suite

Systems Manager contains several tools that solve different operational tasks.

## Session Manager

**Session Manager = secure shell access to EC2 without SSH.**

You don't need:

* a bastion host
* inbound port 22
* SSH keys for the Session Manager session

```text
Admin
  ↓
Session Manager
  ↓
Private EC2
```

### Signal

> **Secure access to private EC2 without SSH → Session Manager**

---

## Run Command

**Run Command = execute commands/scripts on many instances.**

```text
500 EC2 instances
      ↓
  Run Command
      ↓
Run the same script
```

### Signal

> **Run a command across many EC2 instances → Run Command**

---

## Patch Manager

**Patch Manager = automate OS patching.**

Example:

> Apply security patches to hundreds of EC2 instances.

→ **Patch Manager**

### Memory

```text
Session Manager
= ACCESS instances

Run Command
= RUN commands

Patch Manager
= PATCH instances
```

---

## Hybrid Systems Manager

Systems Manager can also manage supported on-premises servers when the SSM Agent and required connectivity are configured.

### Signal

> **Manage servers across AWS and on-premises → Systems Manager**

---

# Developer & Application Services

## AWS Batch

**AWS Batch = run batch computing jobs without managing your own job scheduler.**

Example:

```text
Thousands of jobs
      ↓
AWS Batch
      ↓
Compute capacity
```

It is suitable for jobs that may run for a long time and need batch scheduling.

### Signal

> **Long-running batch jobs → AWS Batch**

Comparison:

```text
Lambda
= short event-driven functions

AWS Batch
= batch computing workloads
```

---

## AppSync

**AWS AppSync = managed GraphQL API service.**

Use it for:

* GraphQL
* real-time subscriptions
* application data synchronization

### Signal

> **GraphQL → AppSync**

---

## Amplify

**AWS Amplify = tools for quickly building and deploying web/mobile applications.**

It helps frontend developers connect applications to AWS backend capabilities.

### Signal

> **Quick full-stack web/mobile development → Amplify**

---

## SES

**Amazon SES = send application email.**

Examples:

* verification emails
* receipts
* notifications
* marketing emails

### Signal

> **Application needs to send email → SES**

Don't confuse:

```text
SNS
= notifications / pub-sub

SES
= email
```

---

## S3 Batch Operations

**S3 Batch Operations = perform operations on many existing S3 objects.**

Examples:

* copy objects
* restore objects
* add tags
* invoke Lambda
* other supported object operations

Example:

```text
Millions of existing objects
        ↓
S3 Batch Operations
        ↓
Apply operation
```

### Important idea

A bucket's new default-encryption setting does **not retroactively modify old objects**.

If you need to perform an operation across huge numbers of existing objects:

→ **S3 Batch Operations**

### Signal

> **Mass operation on existing S3 objects → S3 Batch Operations**

---

## Aurora Cloning

**Aurora Cloning = quickly create an Aurora database copy using copy-on-write.**

Useful for:

* testing
* development
* experiments

You don't immediately need a full independent copy of all the storage.

### Signal

> **Quick Aurora copy for testing → Aurora Cloning**

---

## X-Ray

**AWS X-Ray = distributed tracing.**

It follows a request across multiple services.

```text
Client
  ↓
API Gateway
  ↓
Lambda
  ↓
Service A
  ↓
Service B
```

X-Ray helps identify:

* where a request is slow
* where a request failed
* which service causes latency

### Important distinction

```text
CloudWatch
= metrics + logs + monitoring

X-Ray
= distributed request tracing
```

### Signal

> **Find which microservice is causing latency → X-Ray**

---

## AWS Artifact

**AWS Artifact = access AWS compliance reports and agreements.**

Examples include AWS compliance documentation such as:

* SOC reports
* PCI reports
* ISO-related reports

### Signal

> **Auditor needs AWS compliance documents → Artifact**

Artifact is a **document portal**, not a monitoring service.

---

# Common Question Patterns

> **Existing RabbitMQ application + minimal code changes**
> → **Amazon MQ**

> **Move existing servers to AWS with minimal application changes**
> → **MGN**

> **Disaster recovery for physical/virtual/cloud servers**
> → **AWS DRS**

> **Migrate Oracle to Aurora PostgreSQL**
> → **SCT + DMS**

> **HTTPS certificate for ALB**
> → **ACM**

> **HTTPS certificate for CloudFront**
> → **ACM in us-east-1**

> **IPv6 instances need outbound Internet access only**
> → **Egress-Only Internet Gateway**

> **On-premises systems need to resolve private AWS DNS names**
> → **Route 53 Resolver inbound endpoint**

> **AWS resources need to resolve on-premises DNS names**
> → **Route 53 Resolver outbound endpoint**

> **AWS infrastructure must run in the company's own data center**
> → **Outposts**

> **Very low latency for users in a specific metropolitan area**
> → **Local Zones**

> **Very low latency through a 5G network**
> → **Wavelength**

> **Serverless ETL + schema discovery + data catalog**
> → **AWS Glue**

> **Discover the schema of files in S3**
> → **Glue Crawler**

> **Store dataset/table metadata**
> → **Glue Data Catalog**

> **Transform CSV to Parquet**
> → **Glue ETL**

> **Managed Apache Spark processing**
> → **Amazon EMR**

> **Existing Apache Kafka workload**
> → **Amazon MSK**

> **SQL directly on S3**
> → **Amazon Athena**

> **Business dashboards**
> → **Amazon QuickSight**

> **Data lake + fine-grained permissions**
> → **Lake Formation**

> **Move Salesforce data to S3 without custom integration**
> → **AppFlow**

> **Extract fields/tables from scanned invoices**
> → **Textract**

> **Speech recordings → text**
> → **Transcribe**

> **Application reads text aloud**
> → **Polly**

> **Sentiment analysis of customer reviews**
> → **Comprehend**

> **Search internal company documents**
> → **Kendra**

> **Personalized recommendations**
> → **Personalize**

> **Predict future demand from historical time-series data**
> → **Forecast**

> **Build a conversational chatbot**
> → **Lex**

> **Train and deploy a custom ML model**
> → **SageMaker**

> **Secure shell access to private EC2 without SSH**
> → **SSM Session Manager**

> **Run the same command on hundreds of EC2 instances**
> → **SSM Run Command**

> **Automate OS patching**
> → **SSM Patch Manager**

> **Long-running batch computing jobs**
> → **AWS Batch**

> **GraphQL API / subscriptions**
> → **AppSync**

> **Quick web/mobile application development**
> → **Amplify**

> **Application needs to send email**
> → **SES**

> **Apply an operation to millions of existing S3 objects**
> → **S3 Batch Operations**

> **Create a fast Aurora copy for testing**
> → **Aurora Cloning**

> **Find which microservice causes latency**
> → **X-Ray**

> **Auditor needs AWS compliance reports**
> → **AWS Artifact**

---

# Pocket Card

| Keyword                                | Answer                           |
| -------------------------------------- | -------------------------------- |
| Existing RabbitMQ / ActiveMQ           | **Amazon MQ**                    |
| Rehost / lift-and-shift servers        | **MGN**                          |
| Server disaster recovery               | **DRS**                          |
| Database migration                     | **DMS**                          |
| Different database engines             | **SCT + DMS**                    |
| TLS/SSL certificates                   | **ACM**                          |
| CloudFront certificate                 | **ACM in us-east-1**             |
| IPv6 outbound-only Internet            | **Egress-Only Internet Gateway** |
| On-prem → AWS DNS                      | **Resolver inbound endpoint**    |
| AWS → on-prem DNS                      | **Resolver outbound endpoint**   |
| AWS infrastructure in your data center | **Outposts**                     |
| Low latency to a city                  | **Local Zones**                  |
| 5G edge                                | **Wavelength**                   |
| Serverless ETL / data catalog          | **Glue**                         |
| Discover schema                        | **Glue Crawler**                 |
| Store metadata                         | **Glue Data Catalog**            |
| Transform data                         | **Glue ETL**                     |
| Spark / Hadoop                         | **EMR**                          |
| Kafka                                  | **MSK**                          |
| SQL on S3                              | **Athena**                       |
| BI dashboards                          | **QuickSight**                   |
| Data lake permissions                  | **Lake Formation**               |
| SaaS → AWS                             | **AppFlow**                      |
| Images / video                         | **Rekognition**                  |
| Speech → text                          | **Transcribe**                   |
| Text → speech                          | **Polly**                        |
| Translate text                         | **Translate**                    |
| Text / sentiment analysis              | **Comprehend**                   |
| Scanned forms / invoices               | **Textract**                     |
| Enterprise document search             | **Kendra**                       |
| Recommendations                        | **Personalize**                  |
| Time-series forecasting                | **Forecast**                     |
| Chatbot                                | **Lex**                          |
| Custom ML model                        | **SageMaker**                    |
| Secure instance access                 | **SSM Session Manager**          |
| Run commands on many instances         | **SSM Run Command**              |
| OS patching                            | **SSM Patch Manager**            |
| Batch computing                        | **AWS Batch**                    |
| GraphQL                                | **AppSync**                      |
| Web/mobile app development             | **Amplify**                      |
| Application email                      | **SES**                          |
| Bulk S3 object operations              | **S3 Batch Operations**          |
| Quick Aurora copy                      | **Aurora Cloning**               |
| Distributed tracing                    | **X-Ray**                        |
| Compliance documents                   | **AWS Artifact**                 |

# Final Memory

```text
Amazon MQ
= EXISTING MESSAGE BROKER

MGN
= MOVE SERVERS

DRS
= RECOVER SERVERS

DMS
= MOVE DATABASE DATA

SCT
= CONVERT DATABASE SCHEMA

ACM
= HTTPS CERTIFICATES

Egress-Only IGW
= IPv6 OUTBOUND ONLY

Outposts
= AWS IN YOUR DATA CENTER

Local Zones
= AWS CLOSER TO A CITY

Wavelength
= AWS ON 5G

Glue
= ETL + DATA CATALOG

Glue Crawler
= DISCOVER SCHEMA

Glue Data Catalog
= STORE METADATA

Glue ETL
= TRANSFORM DATA

EMR
= SPARK / HADOOP

MSK
= KAFKA

Athena
= SQL ON S3

Lake Formation
= DATA LAKE PERMISSIONS

AppFlow
= SAAS → AWS

Textract
= SCANNED DOCUMENTS

Kendra
= DOCUMENT SEARCH

SSM Session Manager
= SECURE INSTANCE ACCESS

SSM Run Command
= RUN COMMANDS

SSM Patch Manager
= PATCH INSTANCES

Batch
= BATCH COMPUTING

AppSync
= GRAPHQL

SES
= EMAIL

X-Ray
= DISTRIBUTED TRACING

Artifact
= COMPLIANCE DOCUMENTS
```

## The golden rule

```text
Don't memorize the implementation.

Memorize the unique signal.
```

For example:

```text
RabbitMQ       → MQ
Rehost         → MGN
Disaster       → DRS
Kafka          → MSK
Spark          → EMR
5G             → Wavelength
Discover schema→ Glue Crawler
Store metadata → Glue Data Catalog
Transform data → Glue ETL
Scanned form   → Textract
GraphQL        → AppSync
Compliance     → Artifact
```
