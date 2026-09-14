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

Apache Flink / real-time stream processing
→ Managed Service for Apache Flink
```

---

# Migration & Messaging

## Amazon MQ

**Amazon MQ = managed message broker for applications that already use traditional messaging systems.**

It supports managed brokers such as:

* ActiveMQ
* RabbitMQ

It is useful when an existing application already uses traditional messaging protocols or APIs and you want to migrate it to AWS without rewriting the application.

Common keywords:

* RabbitMQ
* ActiveMQ
* JMS
* AMQP
* existing message broker
* minimal application changes

### Important distinction

```text
Existing broker-based application
→ Amazon MQ

New AWS-native application
→ SQS / SNS / EventBridge
```

### Example

> "An existing application uses RabbitMQ and the company wants to migrate to AWS with minimal code changes."

→ **Amazon MQ**

---

## AWS Application Migration Service (MGN)

**AWS Application Migration Service (MGN) = rehost / lift-and-shift servers to AWS.**

The service continuously replicates the source server's **block-level data** to AWS.

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

The goal is to move the server to AWS with **minimal changes**.

### Use it when

> "Move existing physical or virtual servers to AWS without redesigning the application."

→ **MGN**

MGN provides continuous data protection with recovery points near seconds and can achieve recovery in minutes in appropriate configurations.

### Remember

> **Rehost / lift-and-shift → MGN**

---

## AWS Elastic Disaster Recovery (DRS)

**AWS Elastic Disaster Recovery = disaster recovery for servers.**

It continuously replicates workloads into AWS, but the recovery environment is mainly used **when a disaster happens**.

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

### Use it when

> "We need a cost-effective disaster recovery solution for physical, virtual, or cloud servers."

→ **AWS DRS**

### MGN vs DRS

```text
MGN
= migrate to AWS

DRS
= recover in AWS when disaster happens
```

Both use continuous replication, but the goal is different.

---

## AWS DMS + SCT

**AWS Database Migration Service (DMS) = move or replicate database data.**

**AWS Schema Conversion Tool (SCT) = convert schema/code when changing database engines.**

```text
Oracle
   ↓
SCT → convert schema
   ↓
DMS → move data
   ↓
Aurora PostgreSQL
```

### Remember

> **DMS = move data**

> **SCT = convert schema**

### Important pattern

```text
Same database engine
→ DMS

Different database engine
→ SCT + DMS
```

---

# The 7 Rs of migration

The 7 Rs describe different migration strategies.

| R                           | Meaning                    | Simple idea                            |
| --------------------------- | -------------------------- | -------------------------------------- |
| **Rehost**                  | Move without major changes | Lift and shift                         |
| **Replatform**              | Move with small changes    | Use a managed AWS service              |
| **Repurchase**              | Replace the application    | Buy a SaaS product                     |
| **Refactor / Re-architect** | Redesign the application   | Build it for cloud-native architecture |
| **Relocate**                | Move the whole environment | VMware Cloud on AWS, for example       |
| **Retain**                  | Keep it where it is        | Don't migrate yet                      |
| **Retire**                  | Stop using it              | Decommission it                        |

### Easy examples

```text
Rehost
On-prem VM → EC2

Replatform
MySQL on EC2 → Amazon RDS

Repurchase
Self-hosted CRM → SaaS CRM

Refactor
Monolith → Lambda / containers / serverless architecture

Relocate
VMware environment → VMware Cloud on AWS

Retain
Keep the workload on-premises for now

Retire
Delete an unused application
```

---

# Certificates & network services

## ACM — AWS Certificate Manager

**ACM = TLS/SSL certificates for AWS services.**

Use it when you need HTTPS.

Common integrations:

* ALB
* CloudFront
* API Gateway

### Important facts

* Public ACM certificates are provided at no additional charge.
* ACM can automatically renew certificates that meet the renewal requirements.
* A CloudFront certificate must be in **us-east-1**.
* A regional service such as an ALB normally uses a certificate in the same Region.

### Important exam trap

You generally cannot export an **ACM public certificate's private key** for installation on an EC2 server.

So:

```text
HTTPS on ALB
→ ACM certificate in the SAME REGION as the ALB

HTTPS on CloudFront
→ ACM in us-east-1
```

---

# Edge & hybrid infrastructure

| Service         | What it does                                                       | Signal keyword                 |
| --------------- | ------------------------------------------------------------------ | ------------------------------ |
| **Outposts**    | AWS infrastructure installed in your own data center               | AWS services **on-premises**   |
| **Local Zones** | AWS infrastructure closer to users in a specific metropolitan area | Very low latency to a **city** |
| **Wavelength**  | AWS infrastructure inside telecom 5G networks                      | **5G / mobile edge**           |

## Outposts

**AWS Outposts = AWS infrastructure physically installed in your data center.**

Use it when:

* data must remain on-premises
* you need very low latency to local systems
* local workloads must use AWS APIs/services

```text
Your data center
       ↓
   AWS Outposts
       ↓
AWS-style infrastructure
```

### Signal

> **AWS infrastructure on-premises → Outposts**

---

## Local Zones

**Local Zones = AWS infrastructure placed closer to users in a metropolitan area.**

Use it when an application needs very low latency for users in a specific city.

> **City-level low latency → Local Zones**

---

## Wavelength

**AWS Wavelength = AWS infrastructure inside telecom 5G networks.**

Use it for:

* mobile applications
* 5G applications
* extremely low-latency mobile workloads

> **5G → Wavelength**

---

# Analytics family

| Service                              | What it does                                                | Signal keyword                     |
| ------------------------------------ | ----------------------------------------------------------- | ---------------------------------- |
| **AWS Glue**                         | Serverless ETL + Data Catalog                               | ETL / data preparation / catalog   |
| **Glue Crawler**                     | Automatically discovers data and schema                     | Discover schema                    |
| **Glue Data Catalog**                | Stores metadata about datasets                              | Metadata / tables / schema         |
| **Glue ETL**                         | Performs data transformations                               | CSV → Parquet / ETL                |
| **EMR**                              | Managed big-data processing                                 | Spark / Hadoop                     |
| **Managed Service for Apache Flink** | Managed real-time stream processing                         | Apache Flink / real-time streaming |
| **Flink Studio**                     | Interactive Apache Flink development and streaming analysis | Interactive Flink / streaming SQL  |
| **MSK**                              | Managed Apache Kafka                                        | Kafka                              |
| **Athena**                           | SQL directly on S3                                          | SQL on S3                          |
| **QuickSight**                       | BI dashboards                                               | Business dashboards                |
| **Lake Formation**                   | Build/manage a data lake with fine-grained access control   | Data lake + permissions            |
| **AppFlow**                          | Move data between SaaS and AWS services                     | Salesforce → S3                    |

---

## AWS Glue

**AWS Glue = serverless ETL and Data Catalog service.**

ETL means:

```text
Extract
Transform
Load
```

Glue can:

* discover data
* catalog schemas
* transform data
* prepare data for analytics

### The three Glue pieces you should know

```text
Glue Crawler
= discovers schema

Glue Data Catalog
= stores metadata

Glue ETL
= performs the transformation
```

### Simple example

Suppose CSV files arrive in S3:

```text
S3 source bucket
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

### Glue Crawler

**Glue Crawler automatically discovers the structure/schema of data.**

It can inspect sources such as S3 and determine things like:

```text
customer_id → integer
name        → string
price       → decimal
```

It then updates the **Glue Data Catalog**.

### Signal

> **"Automatically discover the schema of files in S3."**

→ **Glue Crawler**

### Important

> **Crawler discovers. It does not perform the ETL transformation.**

---

## Glue Data Catalog

**Glue Data Catalog = centralized metadata store.**

It stores information about datasets such as:

* table definitions
* columns
* data types
* data locations
* schema

Think:

```text
Actual data
→ S3

Information ABOUT the data
→ Glue Data Catalog
```

### Signal

> **"Store metadata/schema/table definitions."**

→ **Glue Data Catalog**

### Important

> **Data Catalog stores metadata. It does not transform the actual data.**

---

## Glue ETL

**Glue ETL = performs the actual data transformation.**

For example:

```text
CSV
 ↓
Glue ETL
 ↓
Parquet
```

It can perform common transformations such as:

* format conversion
* filtering
* joining
* cleaning
* restructuring

### Signal

> **"Convert CSV files to Parquet."**

→ **Glue ETL**

### Important distinction

```text
Glue Crawler
= discovers schema

Glue Data Catalog
= stores metadata

Glue ETL
= transforms data
```

---

## Glue ETL + S3

A typical serverless solution can look like:

```text
S3
 ↓ Object Created
EventBridge
 ↓
Glue ETL job
 ↓
CSV → Parquet
 ↓
S3 transformed bucket
```

Or the process can use a scheduled Glue job.

Glue is preferred when the requirement is **serverless ETL with low operational overhead**, especially for larger data-processing workloads.

### Common comparison

```text
EC2 + Spark
→ You manage the servers

EMR
→ Managed big-data/Spark infrastructure

Lambda
→ Better for lightweight event-driven processing

Glue
→ Serverless ETL
```

### Common exam trap

> **Glue Crawler discovers/catalogs the data; Glue ETL performs the transformation.**

So:

```text
CSV → Parquet
→ Glue ETL

Discover CSV schema
→ Glue Crawler

Store schema/metadata
→ Glue Data Catalog
```

---

## EMR

**Amazon EMR = managed big-data processing using frameworks such as Apache Spark and Hadoop.**

Use it for:

* large-scale data processing
* Spark jobs
* Hadoop workloads

You can use **Spot Instances for suitable EMR task nodes** to reduce cost.

### BI + standard SQL + analytical workloads

When you see:

> **BI + standard SQL + analytical workloads**

→ **Amazon Redshift** is the typical analytical destination.

EMR can process the large dataset, while **Amazon Redshift** provides the high-performance data warehouse used by BI tools and standard SQL queries.

Typical pattern:

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

### EMR vs Redshift

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

A question can require **both** services:

```text
EMR → process big data
Redshift → analyze the processed data
```

### Example

> "A company stores large datasets in S3 and wants to use big-data processing frameworks to process the data. Business users then need high-performance access using BI tools and standard SQL queries."

→ **Amazon EMR + Amazon Redshift**

### Signal

> **Spark / Hadoop → EMR**

> **BI + standard SQL + analytical workloads → Redshift**

---

## Managed Service for Apache Flink

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

### What is Apache Flink?

**Apache Flink is a distributed stream-processing framework.**

The key idea is:

```text
Incoming events
      ↓
continuous processing
      ↓
results in near real time
```

Instead of waiting for a large batch of data to accumulate, Flink can process events as they arrive.

### Signal

> **Apache Flink + real-time streaming analytics → Managed Service for Apache Flink**

### Example

> "A company receives continuous streaming data from Amazon Kinesis and needs to perform real-time analytics using Apache Flink."

→ **Amazon Managed Service for Apache Flink**

---

## Flink Studio

**Managed Service for Apache Flink Studio = an interactive environment for developing and analyzing Apache Flink streaming applications.**

It is useful when you want to:

* interactively explore streaming data
* run SQL queries against streaming data
* develop Flink applications
* test streaming logic
* analyze streaming data interactively

It provides an interactive development experience rather than requiring you to build everything as a traditional production Flink application first.

Typical idea:

```text
Streaming source
(Kinesis / Kafka)
       ↓
   Flink Studio
       ↓
Interactive queries / analysis
       ↓
Explore streaming data
```

### Signal

> **Interactive Apache Flink development or streaming analysis → Flink Studio**

### Flink Studio vs Managed Service for Apache Flink

Think of them as:

```text
Managed Service for Apache Flink
= run managed Flink stream-processing applications

Flink Studio
= interactively develop / query / analyze Flink streaming workloads
```

### Simple distinction

```text
"Run real-time stream processing with Apache Flink"
→ Managed Service for Apache Flink

"Interactively analyze streaming data using Flink"
→ Flink Studio
```

### Important SAA distinction: Flink vs EMR

```text
Apache Flink
= real-time stream processing

EMR
= managed big-data processing
= commonly Spark / Hadoop
```

Think:

```text
Continuous incoming events
→ Flink

Large-scale Spark / Hadoop processing
→ EMR
```

### Common exam trap

Do not automatically choose EMR just because the question says **big data**.

Look for the framework and processing model:

```text
Apache Spark
→ EMR

Hadoop
→ EMR

Apache Flink
→ Managed Service for Apache Flink

Real-time stream processing
→ Managed Service for Apache Flink

Interactive Flink streaming analysis
→ Flink Studio
```

### Very important keyword mapping

```text
Apache Flink
→ Managed Service for Apache Flink

Flink Studio
→ Interactive Flink analysis / development

Spark
→ EMR

Hadoop
→ EMR

Kafka
→ MSK
```

---

## MSK

**Amazon Managed Streaming for Apache Kafka = managed Kafka.**

You manage Kafka-compatible workloads without managing the Kafka infrastructure yourself.

### Signal

> **Kafka → MSK**

Don't overthink it.

---

## Athena

**Athena = SQL directly on S3.**

You already know this from Section 12.

### Signal

> **SQL + S3 → Athena**

---

## QuickSight

**Amazon QuickSight = business intelligence and dashboards.**

Use it to create:

* dashboards
* charts
* reports
* business visualizations

### Signal

> **Business dashboard → QuickSight**

---

## Lake Formation

**AWS Lake Formation = build and manage a data lake with centralized, fine-grained permissions.**

It can control access to specific:

* tables
* columns
* rows

### Signal

> **Data lake + fine-grained permissions → Lake Formation**

---

## AppFlow

**Amazon AppFlow = transfer data between SaaS applications and AWS services without writing the integration yourself.**

Example:

```text
Salesforce
    ↓
AppFlow
    ↓
S3
```

### Signal

> **Salesforce/SaaS → S3 or Redshift → AppFlow**

---

# ML one-liners

| Service         | What it does                                                                                         | Signal                                 |
| --------------- | ---------------------------------------------------------------------------------------------------- | -------------------------------------- |
| **Rekognition** | Looks at **images and videos** and detects things such as faces, objects, people, and unsafe content | Faces, objects, video                  |
| **Transcribe**  | Takes **audio/speech** and turns it into **written text**                                            | Call recording → transcript            |
| **Polly**       | Takes **written text** and turns it into **spoken audio**                                            | App reads text aloud                   |
| **Translate**   | Takes **text in one language** and translates it into another language                               | English → French                       |
| **Comprehend**  | Takes **text** and analyzes its meaning, such as **sentiment, entities, and key phrases**            | "Is this review positive or negative?" |
| **Textract**    | Takes **scanned documents/images** and extracts **text, tables, and form fields**                    | Invoice/form → structured data         |
| **Kendra**      | Searches **company documents** and finds relevant answers using natural-language queries             | "Find our vacation policy"             |
| **Personalize** | Uses user/item behavior to generate **personalized recommendations**                                 | "Customers also bought..."             |
| **Forecast**    | Uses historical **time-series data** to predict future values                                        | Predict future sales/demand            |
| **Lex**         | Lets you build **conversational chatbots** that understand user messages and respond                 | "Build a customer-service chatbot"     |
| **SageMaker**   | Lets data scientists **build, train, tune, and deploy their own ML models**                          | Train your own ML model                |

### Picture / video

→ **Rekognition**

### Audio

→ **Transcribe**

```text
Audio
 ↓
Transcribe
 ↓
Text
```

### Text → speech

→ **Polly**

### Text → another language

→ **Translate**

### Understand text

→ **Comprehend**

### Scanned document → text/tables/forms

→ **Textract**

### Search company documents

→ **Kendra**

### Recommend products/content

→ **Personalize**

### Predict future numbers

→ **Forecast**

### Chat with users

→ **Lex**

### Build your own ML model

→ **SageMaker**

### The important distinctions

```text
Image / video
→ Rekognition

Speech → text
→ Transcribe

Text → speech
→ Polly

Text → another language
→ Translate

Text meaning / sentiment
→ Comprehend

Scanned document → structured information
→ Textract

Search company documents
→ Kendra

Recommendations
→ Personalize

Time-series forecasting
→ Forecast

Chatbot
→ Lex

Build your own ML model
→ SageMaker
```

### Common trap

> **"Extract names, fields, tables, and values from scanned invoices."**

→ **Textract**

Not Rekognition.

```text
Rekognition
= image/video analysis

Textract
= document/text/table/form extraction
```

---

# Systems Manager (SSM) suite

AWS Systems Manager contains several tools that solve different operational tasks.

## Session Manager

**Session Manager = secure shell access to EC2 without SSH.**

You don't need:

* a bastion host
* an open inbound port 22
* SSH keys for the session itself

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

Example:

```text
500 EC2 instances
      ↓
   Run Command
      ↓
Run the same script
on all selected instances
```

### Signal

> **Run a command across many EC2 instances → Run Command**

---

## Patch Manager

**Patch Manager = automate OS patching.**

Example:

> "Apply security patches to 500 EC2 instances every month."

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

Systems Manager can also manage supported **on-premises servers** when the SSM Agent and required connectivity are configured.

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

# Developer & app services

## AWS Batch

**AWS Batch = run large batch jobs without managing the job scheduler yourself.**

It is designed for jobs that may run for a long time.

Example:

```text
Thousands of jobs
      ↓
AWS Batch
      ↓
EC2 / Spot capacity
```

### Signal

> **Long-running batch jobs / containers → AWS Batch**

A common comparison:

```text
Lambda
= short event-driven functions

AWS Batch
= long-running batch workloads
```

---

## AppSync

**AWS AppSync = managed GraphQL API service.**

Use it when the question says:

* GraphQL
* real-time subscriptions
* offline synchronization for applications

Example:

```text
Mobile/Web app
      ↓
   AppSync
      ↓
Data sources
```

### Signal

> **GraphQL → AppSync**

---

## Amplify

**AWS Amplify = tools for quickly building and deploying web/mobile applications.**

It helps developers connect frontend applications with AWS backend services.

### Signal

> **Quick full-stack web/mobile development → Amplify**

---

## SES

**Amazon SES (Simple Email Service) = send application emails.**

Examples:

* receipts
* verification emails
* notifications
* marketing emails

### Signal

> **Application needs to send email → SES**

Don't confuse this with SNS:

```text
SNS
= notifications/pub-sub

SES
= email sending
```

---

## S3 Batch Operations

**S3 Batch Operations = perform an operation on many existing S3 objects.**

For example:

* copy objects
* restore objects
* add tags
* invoke Lambda
* perform other supported object-level operations

Example:

```text
Millions of existing objects
        ↓
S3 Batch Operations
        ↓
Apply the operation
```

### Important idea

A bucket's default encryption setting does **not retroactively change old objects**.

So:

> **"Perform an operation on millions/billions of existing S3 objects."**

→ **S3 Batch Operations**

---

## X-Ray

**AWS X-Ray = distributed tracing.**

It follows a request as it moves through different services.

Example:

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

* which service is slow
* where a request failed
* where latency is coming from

### Important distinction

```text
CloudWatch
= metrics + logs + monitoring

X-Ray
= trace one request across services
```

### Signal

> **Find which microservice is causing latency → X-Ray**

---

## AWS Artifact

**AWS Artifact = access AWS compliance documents and reports.**

Examples include AWS compliance documentation such as:

* SOC
* PCI
* ISO-related reports

### Signal

> **Auditor needs AWS compliance reports → Artifact**

Artifact is a document portal. It is not a monitoring service.

---

## AppStream 2.0

**Amazon AppStream 2.0 = stream desktop applications to users from AWS.**

The application runs on AWS, while the user accesses it remotely through a browser or compatible client.

```text
User's laptop
     ↓
Browser
     ↓
AppStream 2.0
     ↓
Application runs on AWS
```

---

# Question patterns

> **"Existing on-premises application uses RabbitMQ and must migrate with minimal code changes."**
> → **Amazon MQ**

> **"Move existing servers to AWS with minimal application changes."**
> → **AWS Transform MGN**

> **"Need disaster recovery for physical/virtual/cloud servers."**
> → **AWS Elastic Disaster Recovery (DRS)**

> **"Migrate Oracle to Aurora PostgreSQL."**
> → **AWS SCT + DMS**

> **"Need HTTPS certificate for CloudFront."**
> → **ACM in us-east-1**

> **"IPv6 instances need outbound Internet access but must block unsolicited inbound connections."**
> → **Egress-Only Internet Gateway**

> **"On-premises servers need to resolve private AWS DNS names."**
> → **Route 53 Resolver inbound endpoint**

> **"AWS resources need to resolve internal corporate DNS names."**
> → **Route 53 Resolver outbound endpoint**

> **"Workloads must run in the company's own data center but use AWS infrastructure/services."**
> → **Outposts**

> **"Need very low latency for users in a specific metropolitan area."**
> → **Local Zones**

> **"Application needs extremely low-latency processing over a 5G network."**
> → **Wavelength**

> **"Serverless ETL and a central data catalog are required."**
> → **AWS Glue**

> **"Automatically discover the schema of data files in S3."**
> → **Glue Crawler**

> **"Store table/schema metadata for analytics."**
> → **Glue Data Catalog**

> **"Convert CSV files into Parquet."**
> → **Glue ETL**

> **"Managed Apache Spark processing."**
> → **Amazon EMR**

> **"Process continuous streaming data using Apache Flink."**
> → **Managed Service for Apache Flink**

> **"Interactively query or analyze streaming data using Apache Flink."**
> → **Flink Studio**

> **"Existing Apache Kafka workload."**
> → **Amazon MSK**

> **"Query files in S3 using SQL."**
> → **Amazon Athena**

> **"Business users need dashboards and visual reports."**
> → **Amazon QuickSight**

> **"Build a data lake with fine-grained table/row/column permissions."**
> → **Lake Formation**

> **"Move Salesforce data to S3 without custom integration code."**
> → **AppFlow**

> **"Automatically extract fields and tables from scanned invoices."**
> → **Textract**

> **"Search internal company documents using natural-language queries."**
> → **Kendra**

> **"Speech recordings need to become text."**
> → **Transcribe**

> **"Application needs to read text aloud."**
> → **Polly**

> **"Customer reviews need sentiment analysis."**
> → **Comprehend**

> **"Product recommendations based on user behavior."**
> → **Personalize**

> **"Predict future demand based on historical time-series data."**
> → **Forecast**

> **"Build a conversational chatbot."**
> → **Lex**

> **"Data scientists need to build/train/deploy a custom ML model."**
> → **SageMaker**

> **"Secure shell access to private EC2 without SSH or a bastion."**
> → **SSM Session Manager**

> **"Run the same command on hundreds of EC2 instances."**
> → **SSM Run Command**

> **"Automatically patch hundreds of EC2 instances."**
> → **SSM Patch Manager**

> **"Run long-running batch workloads."**
> → **AWS Batch**

> **"Application needs GraphQL and real-time subscriptions."**
> → **AppSync**

> **"Quickly build and deploy a web/mobile application."**
> → **Amplify**

> **"Application needs to send emails."**
> → **SES**

> **"Apply an operation to millions of existing S3 objects."**
> → **S3 Batch Operations**

> **"Create a fast copy of an Aurora database for testing."**
> → **Aurora Cloning**

> **"Find which microservice is causing latency in a request."**
> → **X-Ray**

> **"Auditors need AWS compliance reports."**
> → **AWS Artifact**

> **"Users need to access a Windows application without installing it locally."**
> → **AWS AppStream 2.0**

---

# Pocket card

| Keyword                                  | Answer                                  |
| ---------------------------------------- | --------------------------------------- |
| Existing RabbitMQ / ActiveMQ application | **Amazon MQ**                           |
| Minimal-change server migration / rehost | **AWS Transform MGN**                   |
| Disaster recovery for servers            | **AWS Elastic Disaster Recovery (DRS)** |
| Database migration                       | **DMS**                                 |
| Different database engines               | **SCT + DMS**                           |
| TLS/SSL certificates                     | **ACM**                                 |
| CloudFront certificate                   | **ACM in us-east-1**                    |
| IPv6 outbound-only Internet              | **Egress-Only Internet Gateway**        |
| On-prem → AWS DNS queries                | **Route 53 Resolver inbound endpoint**  |
| AWS → on-prem DNS queries                | **Route 53 Resolver outbound endpoint** |
| AWS infrastructure in your data center   | **Outposts**                            |
| Low latency to a specific city           | **Local Zones**                         |
| 5G edge                                  | **Wavelength**                          |
| Serverless ETL / data catalog            | **Glue**                                |
| Discover schema                          | **Glue Crawler**                        |
| Store metadata                           | **Glue Data Catalog**                   |
| Transform data                           | **Glue ETL**                            |
| Spark / Hadoop                           | **EMR**                                 |
| Apache Flink / real-time streaming       | **Managed Service for Apache Flink**    |
| Interactive Flink streaming analysis     | **Flink Studio**                        |
| Kafka                                    | **MSK**                                 |
| SQL on S3                                | **Athena**                              |
| BI dashboards                            | **QuickSight**                          |
| Data lake + fine-grained permissions     | **Lake Formation**                      |
| SaaS → S3 / Redshift                     | **AppFlow**                             |
| Images/video                             | **Rekognition**                         |
| Speech → text                            | **Transcribe**                          |
| Text → speech                            | **Polly**                               |
| Text → another language                  | **Translate**                           |
| Text/sentiment analysis                  | **Comprehend**                          |
| Scanned forms/invoices                   | **Textract**                            |
| Enterprise document search               | **Kendra**                              |
| Recommendations                          | **Personalize**                         |
| Time-series forecasting                  | **Forecast**                            |
| Chatbot                                  | **Lex**                                 |
| Custom ML models                         | **SageMaker**                           |
| Secure instance shell without SSH        | **SSM Session Manager**                 |
| Run commands across instances            | **SSM Run Command**                     |
| Automated OS patching                    | **SSM Patch Manager**                   |
| Long-running batch jobs                  | **AWS Batch**                           |
| GraphQL / subscriptions                  | **AppSync**                             |
| Fast web/mobile application development  | **Amplify**                             |
| Application email                        | **SES**                                 |
| Bulk operations on existing S3 objects   | **S3 Batch Operations**                 |
| Quick Aurora copy                        | **Aurora Cloning**                      |
| Distributed request tracing              | **X-Ray**                               |
| AWS compliance reports                   | **AWS Artifact**                        |
| Stream desktop applications              | **AppStream 2.0**                       |

# Final memory

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

Managed Service for Apache Flink
= REAL-TIME STREAM PROCESSING

Flink Studio
= INTERACTIVE FLINK STREAM ANALYSIS

MSK
= KAFKA

Athena
= SQL ON S3

QuickSight
= BI DASHBOARDS

Lake Formation
= DATA LAKE PERMISSIONS

AppFlow
= SAAS → AWS

Rekognition
= IMAGE / VIDEO ANALYSIS

Transcribe
= SPEECH → TEXT

Polly
= TEXT → SPEECH

Translate
= TEXT → ANOTHER LANGUAGE

Comprehend
= UNDERSTAND TEXT

Textract
= SCANNED DOCUMENTS

Kendra
= DOCUMENT SEARCH

Personalize
= RECOMMENDATIONS

Forecast
= TIME-SERIES FORECASTING

Lex
= CHATBOT

SageMaker
= CUSTOM ML

SSM Session Manager
= SECURE INSTANCE ACCESS

SSM Run Command
= RUN COMMANDS ON MANY INSTANCES

SSM Patch Manager
= PATCH INSTANCES

Batch
= BATCH COMPUTING

AppSync
= GRAPHQL

Amplify
= WEB / MOBILE APP DEVELOPMENT

SES
= EMAIL

S3 Batch Operations
= BULK S3 OBJECT OPERATIONS

Aurora Cloning
= FAST AURORA COPY

X-Ray
= DISTRIBUTED TRACING

Artifact
= COMPLIANCE DOCUMENTS

AppStream 2.0
= STREAM DESKTOP APPLICATIONS
```

## The golden rule

```text
Don't memorize the implementation.

Memorize the unique signal.
```

For example:

```text
RabbitMQ        → MQ
Rehost          → MGN
Disaster        → DRS
Kafka           → MSK
Spark           → EMR
Hadoop          → EMR
Apache Flink    → Managed Service for Apache Flink
Flink Studio    → Interactive Flink analysis
5G              → Wavelength
Discover schema → Glue Crawler
Store metadata  → Glue Data Catalog
Transform data  → Glue ETL
Scanned form    → Textract
GraphQL         → AppSync
Compliance      → Artifact
```
