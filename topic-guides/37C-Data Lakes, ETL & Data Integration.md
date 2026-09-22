# Section 37C: Data Lakes, ETL & Data Integration

## The idea

These are AWS services that commonly appear in SAA questions involving **data lakes, ETL, metadata, schema discovery, data integration, and bulk operations on S3 objects**.

You generally don't need deep knowledge of each one.

The best strategy is:

> **Read the requirement → identify the unique keyword → choose the service.**

For example:

```text
Serverless ETL + data catalog
→ AWS Glue

Automatically discover schema in S3
→ Glue Crawler

Store table/schema metadata
→ Glue Data Catalog

Transform CSV → Parquet
→ Glue ETL

Data lake + fine-grained permissions
→ Lake Formation

Salesforce → S3
→ AppFlow

Find/subscribe to third-party datasets
→ AWS Data Exchange

Bulk operation on millions of existing S3 objects
→ S3 Batch Operations
```

---

# AWS Glue

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

It is especially useful when the requirement is **serverless ETL with low operational overhead**, especially for larger data-processing workloads.

### Main Glue components

```text
Glue Crawler
= discovers schema

Glue Data Catalog
= stores metadata

Glue ETL
= performs data transformation
```

### Mental model

```text
             AWS Glue
                │
      ┌─────────┼─────────┐
      ↓         ↓         ↓
   Crawler    Catalog     ETL
  discover    metadata   transform
```

---

# Glue Crawler

**Glue Crawler = automatically discovers the structure/schema of data.**

It can inspect data sources such as S3 and determine things like:

```text
customer_id → integer
name        → string
price       → decimal
```

It then updates the **Glue Data Catalog**.

### Typical flow

```text
S3 files
   ↓
Glue Crawler
   ↓
Discover schema
   ↓
Glue Data Catalog
```

### Signal

> **"Automatically discover the schema of files in S3."**

→ **Glue Crawler**

### Important

> **Crawler discovers. It does not perform the ETL transformation.**

For example, a crawler can determine that a CSV contains:

```text
customer_id
name
price
```

But it does not perform:

```text
CSV → Parquet
```

That is the job of Glue ETL.

---

# Glue Data Catalog

**Glue Data Catalog = centralized metadata store.**

It stores information **about** datasets rather than the actual dataset itself.

It can store:

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

# Glue ETL

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

# Complete Glue Example

Suppose CSV files arrive in an S3 bucket.

A typical serverless data-processing workflow can look like:

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

This separates the three roles:

```text
Crawler
→ Understand the structure

Catalog
→ Remember the structure

ETL
→ Transform the data
```

---

# Glue ETL + S3

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

Or the process can use a **scheduled Glue job**.

Glue is useful when the requirement is:

* serverless ETL
* low operational overhead
* processing larger datasets
* preparing data for analytics

---

# Glue vs EC2 + Spark vs EMR vs Lambda

A common SAA question may give several ways to process data.

### EC2 + Spark

```text
EC2 + Spark
→ You manage the servers
```

You are responsible for more infrastructure management.

### EMR

```text
EMR
→ Managed big-data / Spark infrastructure
```

Amazon EMR is covered more deeply in the analytics section.

### Lambda

```text
Lambda
→ Better for lightweight event-driven processing
```

Lambda can be useful for smaller, short-running transformations triggered by events.

### Glue

```text
Glue
→ Serverless ETL
```

Glue is the natural choice when the question emphasizes **serverless ETL** rather than generic compute.

---

# AWS Glue Service Comparison

| Glue component        | Main job        | Signal                        |
| --------------------- | --------------- | ----------------------------- |
| **Glue Crawler**      | Discover schema | Automatically discover schema |
| **Glue Data Catalog** | Store metadata  | Tables / schema / metadata    |
| **Glue ETL**          | Transform data  | CSV → Parquet / ETL           |

### Quick memory

```text
Discover
→ Crawler

Store
→ Catalog

Transform
→ ETL
```

---

# AWS Lake Formation

**AWS Lake Formation = build and manage a data lake with centralized, fine-grained permissions.**

It helps organize and secure data lakes and can provide fine-grained access control.

It can control access to specific:

* tables
* columns
* rows

### Signal

> **Data lake + fine-grained permissions → Lake Formation**

---

## Example

Suppose a company has a large data lake in S3.

Different users need different access:

```text
Finance team
→ full access to financial tables

Sales team
→ selected columns

Regional managers
→ only rows for their region
```

The requirement is not simply "store data in S3."

It is:

> **Build/manage a data lake with fine-grained permissions.**

→ **AWS Lake Formation**

---

## Lake Formation vs Glue

These two services can appear together.

```text
Glue
= discover / catalog / transform data

Lake Formation
= govern and control access to the data lake
```

### Mental model

```text
AWS Glue
→ Understand and prepare the data

Lake Formation
→ Govern and secure the data lake
```

---

# Amazon AppFlow

**Amazon AppFlow = transfer data between SaaS applications and AWS services without writing the integration yourself.**

It is designed for data integration between supported applications and AWS destinations/sources.

Examples include:

```text
Salesforce
    ↓
AppFlow
    ↓
S3
```

or:

```text
Salesforce
    ↓
AppFlow
    ↓
Amazon Redshift
```

### Signal

> **Salesforce/SaaS → S3 or Redshift → AppFlow**

---

## Example

> "A company wants to move Salesforce data into Amazon S3 without developing custom integration code."

→ **Amazon AppFlow**

### Important idea

AppFlow is about **data transfer/integration between applications and AWS services**.

It is not primarily an ETL engine like Glue.

```text
AppFlow
= move data between SaaS and AWS

Glue
= discover / catalog / transform data
```

---

# AWS Data Exchange

**AWS Data Exchange = discover, subscribe to, share, and use third-party datasets in AWS.**

It provides a way for data providers to make datasets available and for data recipients to discover and subscribe to them. Data products can be available through **AWS Marketplace**, and supported dataset types include files, APIs, Amazon S3, Amazon Redshift, and AWS Lake Formation data.

Typical use case:

```text
Third-party data provider
          ↓
   AWS Data Exchange
          ↓
      Subscriber
          ↓
   AWS analytics / ML
```

Example:

> "A company wants to find and subscribe to a third-party dataset for use in its analytics workload."

→ **AWS Data Exchange**

### Signal

> **Find / subscribe to third-party datasets → AWS Data Exchange**

### Important distinction

```text
AppFlow
= move data between supported SaaS applications and AWS

Data Exchange
= discover / subscribe to external datasets
```

AWS Data Exchange can provide data through several forms, including files, APIs, S3 data, Redshift data, and Lake Formation data.

---

# S3 Batch Operations

**S3 Batch Operations = perform an operation on many existing S3 objects.**

For example, it can perform supported operations such as:

* copy objects
* restore objects
* add tags
* invoke Lambda
* perform other supported object-level operations

The key idea is **bulk operations on existing objects**.

### Example

```text
Millions of existing objects
        ↓
S3 Batch Operations
        ↓
Apply the operation
```

---

# Important S3 Encryption Trap

A bucket's default encryption setting does **not retroactively change old objects**.

Suppose:

```text
S3 bucket
↓
Millions of existing objects
```

You change the bucket's default encryption configuration.

That does not automatically reprocess all existing objects.

When the question instead asks you to:

> **Perform an operation on millions/billions of existing S3 objects**

→ **S3 Batch Operations**

---

## Example

> "A company has millions of existing S3 objects and needs to apply an operation to all of them."

→ **S3 Batch Operations**

The exact operation might be:

* copying objects
* tagging objects
* restoring objects
* invoking Lambda
* another supported object-level operation

The important clue is the **large number of existing objects**.

---

# S3 Batch Operations vs EventBridge / Lambda

Do not confuse bulk processing of existing objects with processing a newly created object.

### New object arrives

```text
S3 Object Created
→ EventBridge / Lambda / other event-driven solution
```

### Millions of existing objects

```text
Existing objects
→ S3 Batch Operations
```

### Mental model

```text
New event
→ Event-driven processing

Huge number of existing objects
→ S3 Batch Operations
```

---

# Data Lake Architecture Example

A larger architecture can combine several services:

```text
                    SaaS applications
                         │
                         ↓
                      AppFlow
                         │
                         ↓
S3 data lake ──────────────────────────────────┐
   │                                            │
   ↓                                            ↓
Glue Crawler                             Lake Formation
   │                                      permissions
   ↓                                            │
Glue Data Catalog                              │
   │                                            │
   ↓                                            │
Glue ETL ───────────────→ transformed data ←───┘
```

The roles remain different:

```text
AppFlow
= bring data from SaaS

S3
= store the data

Glue Crawler
= discover schema

Glue Data Catalog
= store metadata

Glue ETL
= transform data

Lake Formation
= control data lake access
```

---

# Common Question Patterns

> **"Serverless ETL and a central data catalog are required."**

→ **AWS Glue**

---

> **"Automatically discover the schema of data files in S3."**

→ **Glue Crawler**

---

> **"Store table/schema metadata for analytics."**

→ **Glue Data Catalog**

---

> **"Convert CSV files into Parquet."**

→ **Glue ETL**

---

> **"A company wants to build a data lake with fine-grained access control over tables, rows, and columns."**

→ **Lake Formation**

---

> **"Move Salesforce data to S3 without custom integration code."**

→ **AppFlow**

---

> **"Transfer data from a SaaS application to Amazon Redshift without building the integration from scratch."**

→ **AppFlow**

---

> **"A company wants to find and subscribe to a third-party dataset for analytics."**

→ **AWS Data Exchange**

---

> **"Apply an operation to millions of existing S3 objects."**

→ **S3 Batch Operations**

---

> **"Millions of existing S3 objects need to be processed in bulk."**

→ **S3 Batch Operations**

---

# Important SAA Traps

## Glue Crawler vs Glue ETL

This is one of the most important distinctions in this group.

```text
Discover schema
→ Glue Crawler

Transform data
→ Glue ETL
```

For example:

```text
"Find out the columns and data types in CSV files."
→ Crawler

"Convert CSV files to Parquet."
→ Glue ETL
```

---

## Glue Data Catalog vs S3

The Catalog does not contain the actual dataset.

```text
Actual files
→ S3

Metadata about those files
→ Glue Data Catalog
```

For example:

```text
S3
= customer.csv

Glue Data Catalog
= customer table
= columns
= data types
= S3 location
= schema
```

---

## Glue vs Lake Formation

These services can work together, but their roles differ.

```text
Glue
= discover + catalog + transform

Lake Formation
= data lake governance + fine-grained permissions
```

---

## AppFlow vs Glue

Both can move or work with data, but the signal is different.

```text
SaaS application → AWS
→ AppFlow

ETL / transformation / data preparation
→ Glue
```

Example:

```text
Salesforce
   ↓
AppFlow
   ↓
S3
   ↓
Glue
   ↓
Transform data
```

---

## AppFlow vs Data Exchange

```text
SaaS application → AWS
→ AppFlow

Third-party dataset → discover/subscribe/use
→ AWS Data Exchange
```

---

## S3 Batch Operations vs normal S3 events

```text
One newly created object
→ event-driven processing

Millions of existing objects
→ S3 Batch Operations
```

---

# Data Processing Decision Tree

When you see a data-related question, first identify the operation.

```text
What does the question want?
          │
          ├── Discover schema
          │      ↓
          │   Glue Crawler
          │
          ├── Store metadata
          │      ↓
          │   Glue Data Catalog
          │
          ├── Transform data
          │      ↓
          │   Glue ETL
          │
          ├── Govern data lake permissions
          │      ↓
          │   Lake Formation
          │
          ├── SaaS → AWS data transfer
          │      ↓
          │   AppFlow
          │
          ├── Find / subscribe to
          │   third-party datasets
          │      ↓
          │   AWS Data Exchange
          │
          └── Bulk operation on existing
              S3 objects
                 ↓
             S3 Batch Operations
```

---

# Pocket Card

| Keyword                                | Answer                  |
| -------------------------------------- | ----------------------- |
| Serverless ETL / data catalog          | **AWS Glue**            |
| Discover schema                        | **Glue Crawler**        |
| Automatically discover S3 file schema  | **Glue Crawler**        |
| Store metadata                         | **Glue Data Catalog**   |
| Store table/schema definitions         | **Glue Data Catalog**   |
| Transform data                         | **Glue ETL**            |
| CSV → Parquet                          | **Glue ETL**            |
| Data lake + fine-grained permissions   | **Lake Formation**      |
| Table/row/column permissions           | **Lake Formation**      |
| SaaS → S3                              | **AppFlow**             |
| SaaS → Redshift                        | **AppFlow**             |
| Find/subscribe to third-party datasets | **AWS Data Exchange**   |
| Bulk operation on existing S3 objects  | **S3 Batch Operations** |
| Millions/billions of existing objects  | **S3 Batch Operations** |

---

# Final Memory

```text
AWS Glue
= SERVERLESS ETL + DATA CATALOG

Glue Crawler
= DISCOVER SCHEMA

Glue Data Catalog
= STORE METADATA

Glue ETL
= TRANSFORM DATA

Lake Formation
= DATA LAKE PERMISSIONS

AppFlow
= SAAS → AWS

AWS Data Exchange
= THIRD-PARTY DATA

S3 Batch Operations
= BULK OPERATIONS ON EXISTING S3 OBJECTS
```

# The Golden Rule

```text
Serverless ETL
→ Glue

Discover schema
→ Glue Crawler

Store metadata
→ Glue Data Catalog

Transform data
→ Glue ETL

Data lake + fine-grained permissions
→ Lake Formation

Salesforce / SaaS → S3 or Redshift
→ AppFlow

Find or subscribe to third-party datasets
→ AWS Data Exchange

Millions of existing S3 objects
→ S3 Batch Operations
```

> **Don't memorize the implementation.**
>
> **Memorize the unique signal.**

For example:

```text
ETL                    → Glue
Discover schema        → Crawler
Store metadata         → Catalog
CSV → Parquet          → Glue ETL
Data lake access       → Lake Formation
Salesforce → S3        → AppFlow
Third-party datasets   → Data Exchange
Bulk S3 operation      → S3 Batch Operations
```
