# Section 4: EBS, EFS & Instance Store

## The idea

AWS provides several storage options for applications running on EC2. The main choices in this section are:

* **EBS** → persistent **block storage** attached to EC2
* **EFS** → shared **file storage** that many Linux-based clients can use at the same time
* **Instance Store** → very fast **local storage** physically attached to the EC2 host, but temporary

---

# EBS — Elastic Block Store

Amazon EBS provides persistent **block storage** for EC2.

An EBS volume behaves like a disk attached to an EC2 instance. The operating system can format it with a filesystem and use it for:

* operating system disks
* databases
* application files
* logs
* persistent application data

## Core EBS rules

### 1. An EBS volume belongs to one Availability Zone

An EBS volume exists in a specific Availability Zone.

```text
Region
│
├── AZ-A
│   └── EBS Volume
│
└── AZ-B
```

You cannot directly attach the AZ-A volume to an EC2 instance in AZ-B.

To move the data to another AZ:

```text
EBS Volume
   ↓
Snapshot
   ↓
Restore snapshot in another AZ
   ↓
New EBS volume
```

The same basic idea applies when moving EBS data between Regions, with the additional step of copying the snapshot to the destination Region.

---

## 2. EBS is persistent

EBS data normally survives when an EC2 instance is stopped.

Whether the volume survives **instance termination** depends on its `DeleteOnTermination` setting.

### Root volume

For a root EBS volume created when the instance is launched:

```text
DeleteOnTermination = true
```

by default.

Therefore:

```text
EC2 terminated
      ↓
Root EBS volume deleted
```

### Additional EBS volumes

Additional volumes generally default to:

```text
DeleteOnTermination = false
```

Therefore they normally survive instance termination.

### Exam clue

> "The root EBS volume must remain after the EC2 instance is terminated."

→ Set:

```text
DeleteOnTermination = false
```

---

# EBS volume attachment

Normally, an EBS volume is attached to **one EC2 instance at a time**.

There is an important exception:

### EBS Multi-Attach

Supported **io1 and io2** volumes can use Multi-Attach.

A Multi-Attach volume can be attached to **up to 16 supported EC2 instances in the same Availability Zone**.

This is designed for applications that are specifically designed to coordinate shared block storage.

It is **not** a normal shared filesystem.

```text
        io2 Multi-Attach
              │
      ┌───────┼───────┐
      ↓       ↓       ↓
     EC2     EC2     EC2
```

Do not confuse this with:

* **EBS Multi-Attach** → shared **block device**
* **EFS** → shared **file system**

### Important Multi-Attach traps

**Only `io1` and `io2` support EBS Multi-Attach.**

```text
gp3    → ❌ Multi-Attach
st1    → ❌
sc1    → ❌
standard/Magnetic → ❌
io1    → ✅
io2    → ✅
```

Multi-Attach does **not** provide multi-AZ resiliency because all attached instances must be in the **same AZ**.

So:

> **"gp3 with Multi-Attach provides multi-AZ resiliency."**

→ **Wrong**

The correct concept is:

> **io1/io2 Multi-Attach → multiple EC2 instances in the same AZ**

### Exam clue

> "Several EC2 instances must access the same block volume in the same AZ, and the application is cluster-aware."

→ **io1/io2 Multi-Attach**

### Small Elastic Volumes note

**Elastic Volumes** can increase the EBS volume size, change the volume type, and adjust supported performance settings, but **cannot shrink an existing EBS volume**.

> **Increase/grow → ✅ | Shrink → ❌**

---

# New EBS volume: format and mount it

A newly created EBS data volume is just a block device. Linux does not automatically turn it into a usable mounted filesystem.

Typical steps are:

```text
Create EBS volume
      ↓
Attach to EC2
      ↓
Format filesystem
      ↓
Mount filesystem
      ↓
Add to /etc/fstab if it should mount automatically after reboot
```

For example:

```bash
mkfs -t xfs /dev/nvme1n1
mkdir /data
mount /dev/nvme1n1 /data
```

The exact device name depends on the instance and operating system.

### Exam clue

> "A newly attached EBS volume is not available under the expected directory."

Think:

**Format it, mount it, and configure `/etc/fstab` if persistent mounting is required.**

---

# EBS performance: IOPS vs Throughput

This is one of the most important EBS concepts.

## IOPS

**IOPS = Input/Output Operations Per Second**

IOPS measures how many individual read/write operations the storage can handle.

Think about workloads that perform many relatively small random operations:

* databases
* transactional systems
* random reads/writes

Typical keyword:

> **high IOPS**

---

## Throughput

Throughput measures how much data can be transferred per second, usually in **MB/s or GB/s**.

Throughput matters for workloads that process large amounts of data sequentially:

* log processing
* ETL
* large file processing
* streaming
* big-data workloads

Typical keyword:

> **high throughput**

### Simple distinction

```text
Database
→ many small/random operations
→ IOPS

Large files
→ large sequential reads/writes
→ Throughput
```

---

# EBS volume types

The important exam categories are:

### SSD

Used for:

* random I/O
* databases
* boot volumes
* latency-sensitive workloads

Main options:

* **gp3**
* **io1**
* **io2**

### HDD

Used primarily for:

* large sequential workloads
* throughput-oriented workloads

Main options:

* **st1**
* **sc1**

Important:

> **st1 and sc1 cannot be used as root/boot volumes.**

### Previous-generation Magnetic

There is also:

* **Magnetic (`standard`)**

It is a **previous-generation EBS volume type** designed for small datasets where data is accessed infrequently and performance is not the primary concern.

For modern workloads, `sc1` is the current EBS choice for **infrequently accessed data where minimizing storage cost is important**.

### Exam trap

Some older practice questions describe:

> **Magnetic = lowest-cost EBS storage for infrequently accessed data**

That is legacy terminology associated with the `standard` / Magnetic volume type.

For current EBS thinking:

```text
Current low-cost infrequently accessed HDD
→ sc1

Legacy / previous-generation infrequent-access EBS
→ Magnetic (standard)
```

Do not confuse the two.

---

# General Purpose SSD — gp3

**gp3** is the general-purpose SSD option and is the normal default choice for many workloads.

It provides:

* **3,000 baseline IOPS**
* **125 MB/s baseline throughput**
* IOPS and throughput can be provisioned **independently of volume size**
* up to **80,000 IOPS**

This last point is very important.

### Example

Suppose an application needs:

```text
10,000 IOPS
100 GB storage
```

With gp3, you do not need to increase the volume size just to obtain more IOPS.

You can keep:

```text
100 GB
10,000 IOPS
```

because performance can be provisioned independently.

### Exam clue

> "The application needs more IOPS without increasing storage capacity."

→ **gp3**

---

# gp2 vs gp3

`gp2` uses a different model.

### gp2

IOPS is tied to the size of the volume:

```text
3 IOPS per GiB
```

with additional burst behavior.

Therefore, increasing the volume size is one way to increase IOPS.

### gp3

IOPS and throughput are independent of volume size.

|                                         | gp2          | gp3            |
| --------------------------------------- | ------------ | -------------- |
| Baseline IOPS model                     | tied to size | 3,000 baseline |
| IOPS independent of size                | ❌            | ✅              |
| Throughput independent of size          | ❌            | ✅              |
| Typical recommended general-purpose SSD | older        | **gp3**        |

### Exam pattern

> "The company uses gp2 and wants more performance without paying for unnecessary storage."

→ **Migrate to gp3**

Do not interpret "gp2 appears in the question" as an automatic answer. The important clue is **needing independent performance configuration or better price/performance**.

---

# Provisioned IOPS SSD — io1 / io2

Use **io1 or io2** when the workload requires **high, predictable IOPS and consistently low latency**, especially when the workload is more demanding than a general-purpose SSD is intended for.

Typical workloads:

* high-performance relational databases
* demanding transactional applications
* high-performance NoSQL databases
* applications with strict I/O requirements

### Key exam wording

> **"Consistent and low-latency performance"**

→ Think **Provisioned IOPS**

> **"I/O-intensive database"**

→ Think **io1/io2**

The important distinction is:

```text
General-purpose SSD
→ gp3

Provisioned, predictable IOPS
→ io1/io2
```

### Exam clue

> "The application requires consistent and low-latency I/O performance and is highly I/O intensive."

→ **io1/io2**

### gp3 vs Provisioned IOPS

Do not automatically choose `io1/io2` simply because the question says "database".

Use the workload requirements:

```text
Normal database / general-purpose
→ gp3

Very high / predictable IOPS
→ io1/io2
```

---

# io2 Block Express

For extremely high-performance workloads, **io2 Block Express** supports up to:

**256,000 IOPS**

and provides very low latency.

### Exam clue

> "256,000 IOPS"
> "sub-millisecond latency"

→ **io2 Block Express**

---

# Throughput-Optimized HDD — st1

Use **st1** for large sequential workloads where throughput matters more than random IOPS.

Typical examples:

* log processing
* big-data workloads
* ETL
* large sequential datasets

### Exam clue

> "Large sequential reads/writes and high throughput at lower cost."

→ **st1**

---

# Cold HDD — sc1

Use **sc1** for data that is accessed infrequently and where minimizing storage cost is the priority.

### Exam clue

> "Infrequently accessed data stored on EBS at the lowest cost."

→ **sc1**

---

# Magnetic — standard

**Magnetic (`standard`)** is a previous-generation EBS volume type.

It is suited for:

* small datasets
* infrequently accessed data
* workloads where performance is not the primary concern

It offers much lower and less consistent performance than modern SSD volume types.

### Important exam wording

A legacy question may describe:

> "Magnetic volumes provide the lowest cost per gigabyte and are ideal for infrequently accessed data."

This is the kind of wording used in older EBS questions.

The key recognition is:

```text
Magnetic
= standard
= previous-generation
= infrequent access
= performance not primary concern
```

Do not confuse it with:

```text
sc1
= current Cold HDD
= infrequent access
= low-cost current-generation EBS
```

### Exam trap

> **"Spot volume"**

→ There is **no EBS volume type called Spot**.

Spot is an EC2 purchasing option, not an EBS storage type.

> **"SR-IOV volume"**

→ There is **no EBS volume type called SR-IOV**.

SR-IOV is a virtualization/I/O technology, not an EBS volume type.

---

# EBS decision guide

```text
What does the workload need?

          ┌─────────────────────────┐
          │ Random I/O / database?  │
          └────────────┬────────────┘
                       │ Yes
                       ↓
                     SSD
                  ┌────┴────┐
                  ↓         ↓
             General     Provisioned
            purpose       IOPS?
                  ↓         ↓
                 gp3     io1/io2
                            │
                            ↓
                    Extreme performance?
                            │
                            ↓
                     io2 Block Express


Large sequential workload?
          │
          ↓
         HDD
       ┌───┴───┐
       ↓       ↓
   Frequent   Infrequent
    access      access
       ↓         ↓
      st1        sc1


Legacy / previous-generation
infrequent-access workload
          ↓
   Magnetic / standard
```

### Modern performance shortcut

```text
General-purpose SSD
→ gp3

Provisioned, predictable low latency
→ io1/io2

Extreme IOPS / 256,000
→ io2 Block Express
```

---

# EBS snapshots

An EBS snapshot is a point-in-time backup of an EBS volume.

Snapshots are stored in AWS-managed storage and are **incremental after the first snapshot**.

This means AWS stores only the blocks that changed since the previous snapshot.

### Example

```text
Day 1 → Snapshot A
Day 2 → Snapshot B
Day 3 → Snapshot C
```

Snapshots B and C only need to account for changed blocks rather than copying the entire volume as a completely new backup each time.

---

## Using an EBS volume while a snapshot is in progress

An EBS volume **can still be used while its snapshot is being created**.

You can continue to:

* read from the volume
* write to the volume
* use the EC2 instance normally
* perform normal volume operations

A snapshot does **not lock the EBS volume**.

```text
EC2
 ↓
EBS Volume
 ├── Read ✅
 ├── Write ✅
 └── Snapshot in progress ✅
```

### Important exam trap

> **"Can the EBS volume be used while the snapshot is in progress?"**

→ **Yes**

It is **not read-only** and does **not** have to wait for the snapshot to finish.

### Simple mental model

```text
Snapshot
= backup being created

EBS volume
= remains available
```

So:

> **EBS snapshot in progress ≠ EBS volume locked**

---

# Cross-Region EBS backup

To protect EBS data in another Region:

```text
EBS volume
    ↓
Snapshot
    ↓
Copy snapshot to another Region
    ↓
Restore volume there if needed
```

### Exam clue

> "Back up EBS data to another AWS Region for disaster recovery."

→ **Create a snapshot and copy it to the destination Region.**

---

# Snapshot Archive

The **EBS Snapshot Archive** tier is intended for snapshots that are rarely accessed.

It reduces storage cost substantially but has a much slower restore process.

Exam clue:

> "Snapshots are rarely restored and minimizing backup storage cost is more important than fast recovery."

→ **Snapshot Archive**

---

# Recycle Bin

Amazon EBS **Recycle Bin** helps protect against accidental deletion of supported resources such as EBS snapshots.

You configure retention rules so that deleted snapshots remain recoverable for a specified period.

### Exam clue

> "Protect accidentally deleted EBS snapshots."

→ **Recycle Bin**

---

# Fast Snapshot Restore

Normally, data restored from an EBS snapshot is loaded on demand.

This can cause initial reads to be slower while blocks are restored.

**Fast Snapshot Restore (FSR)** pre-initializes the restored volume so that it can provide full performance immediately.

Trade-off:

> Faster recovery and immediate performance, but additional cost.

### Exam clue

> "A restored EBS volume must provide full performance immediately with no initial read penalty."

→ **Fast Snapshot Restore**

---

# Data Lifecycle Manager — DLM

**Amazon Data Lifecycle Manager (DLM)** automates EBS snapshot lifecycle operations.

It can automate:

* snapshot creation
* retention
* deletion
* lifecycle policies
* selected cross-Region snapshot-copy workflows

A common pattern is:

```text
Every day
   ↓
Create snapshot
   ↓
Keep last 7
   ↓
Delete older snapshots
```

### Exam clue

> "Automatically create and retain EBS snapshots on a schedule."

→ **Amazon DLM**

DLM is especially useful when the requirement is specifically about **EBS snapshot lifecycle management**.

For broader centralized backup management across many AWS services, consider **AWS Backup**.

---

# EBS encryption (KMS)

Enable EBS encryption when creating the volume.

Encrypted EBS volumes create encrypted snapshots, and encrypted snapshots can be used to create encrypted volumes.

Encryption protects:

* **Data at rest** on the EBS volume
* **Data in transit between the EC2 instance and EBS volume**
* **EBS snapshots**, which are automatically encrypted when created from an encrypted volume
* **Volumes created from encrypted snapshots**, which are automatically encrypted

### Exam rules

| Scenario                                        | Result                    |
| ----------------------------------------------- | ------------------------- |
| Encrypted EBS volume                            | ✅ Data at rest encrypted  |
| Data moving between EC2 and EBS                 | ✅ Encrypted               |
| Snapshot of encrypted EBS volume                | ✅ Automatically encrypted |
| Volume created from encrypted snapshot          | ✅ Automatically encrypted |
| "Only data in the volume is encrypted"          | ❌                         |
| "Snapshot is not automatically encrypted"       | ❌                         |
| "Volume from encrypted snapshot is unencrypted" | ❌                         |

## EBS Encryption by Default

**EBS Encryption by Default** is a **Region-level account setting** that automatically encrypts new EBS volumes created in that Region.

It also causes a new EBS volume restored from an **unencrypted snapshot** to be encrypted automatically.

```text
EBS Encryption by Default
        ↓
      Region
        ↓
New EBS volumes
        ↓
Automatically encrypted
```

> **Encryption by Default = Region-level setting**

It applies to **new** resources; existing unencrypted EBS volumes and snapshots are not automatically encrypted.

## Existing unencrypted volume

You cannot simply toggle an existing unencrypted EBS volume into an encrypted volume in place.

The normal process is:

```text
Unencrypted EBS volume
        ↓
Create snapshot
        ↓
Copy snapshot with encryption enabled
        ↓
Create encrypted EBS volume
        ↓
Attach new volume
```

### Exam clue

> "Encrypt an existing unencrypted EBS volume."

→ **Snapshot → copy with encryption → create new encrypted volume**

---

# EFS — Elastic File System

Amazon EFS is a managed **file system** designed primarily for Linux-based workloads.

**POSIX-compliant storage → EFS** is a POSIX-compliant file system (NFSv4).

Scalable for HPC → EFS scales automatically with workload.

It uses the **NFS protocol** and allows multiple clients to access the same files at the same time.

This makes it fundamentally different from EBS.

```text
             EFS
              │
      ┌───────┼───────┐
      ↓       ↓       ↓
     EC2     EC2     EC2
```

All clients can work with the same files.

---

## EFS vs EBS

| Feature                  | EBS                            | EFS                                      |
| ------------------------ | ------------------------------ | ---------------------------------------- |
| Type                     | Block storage                  | File storage                             |
| Shared by many instances | Normally no                    | Yes                                      |
| Multi-AZ                 | No, volume is AZ-specific      | Yes                                      |
| Typical protocol         | Block device                   | NFS                                      |
| Typical use              | OS, database, application disk | Shared files                             |
| Capacity management      | Provision volume size          | Automatically grows/shrinks              |
| Typical clients          | EC2                            | Multiple EC2 instances / compute clients |

---

# EFS is for shared files

Typical use cases:

* user uploads
* shared application files
* content management systems
* WordPress content
* shared configuration/data across Linux instances
* applications behind an Auto Scaling Group that need common files

### Exam clue

> "Multiple EC2 instances in different AZs need access to the same files."

→ **EFS**

---

# EFS and Windows

EFS is not the normal choice for native Windows file-sharing requirements.

For Windows workloads requiring:

* SMB
* Windows file shares
* Active Directory integration

think:

**FSx for Windows File Server**

### Exam clue

> "Windows + SMB + Active Directory"

→ **FSx for Windows File Server**

Do not choose EFS for this requirement.

---

# EFS storage and cost

EFS automatically grows and shrinks as files are added or removed.

You do not need to pre-provision a fixed storage capacity like an EBS volume.

EFS generally costs more per GB than EBS, but you do not pay for unused provisioned capacity in the same way.

EFS also provides lifecycle management so less frequently accessed files can move to lower-cost storage classes.

---

# EFS performance modes

EFS provides two performance modes:

### General Purpose

The default and recommended choice for most workloads.

### Max I/O

Designed for workloads with very high concurrency where the application can tolerate higher latency.

Exam clue:

> "Thousands of concurrent clients and maximum aggregate throughput is more important than latency."

→ **Max I/O**

For most applications, choose **General Purpose**.

---

# EFS throughput modes

EFS supports multiple throughput modes.

### Elastic

Automatically scales throughput with workload demand.

This is the modern default choice for many workloads.

### Provisioned

Allows you to explicitly configure throughput independently of storage size.

Useful when the required throughput is known and should not depend on the amount of data stored.

### Bursting

Throughput is tied to the amount of data stored and can burst according to the EFS model.

For exam questions, focus mainly on recognizing:

> **Elastic = automatically adapts to workload**

---

# EFS lifecycle management

EFS can automatically move less frequently accessed files to lower-cost storage classes.

Typical lifecycle pattern:

```text
Frequently accessed
        ↓
EFS Standard

Less frequently accessed
        ↓
EFS IA

Very cold data
        ↓
EFS Archive
```

### Exam clue

> "Reduce the cost of files that have not been accessed for a long time."

→ **EFS lifecycle management**

---

# Instance Store

Instance Store provides **local storage physically attached to the EC2 host**.

Because the storage is local:

* latency is very low
* I/O performance can be extremely high
* there is no network hop like EBS

The major problem is:

> **Instance Store is ephemeral.**

You should use it only when the data can be recreated, discarded, or recovered from another source.

---

## Instance Store data persistence

Instance Store data survives an **EC2 reboot**.

It does **not** survive:

* stop
* hibernate
* terminate

So:

```text
Reboot
→ same running host
→ instance-store data survives

Stop
→ instance-store data lost

Terminate
→ instance-store data lost

Hibernate
→ instance-store data lost
```

### Exam clue

> "Temporary cache, scratch space, or data replicated elsewhere."

→ **Instance Store**

---

# When Instance Store is a good choice

Examples:

* temporary files
* scratch space
* caching
* intermediate processing data
* data that is replicated by the application
* high-speed local processing

For example, a Cassandra cluster may replicate data across multiple nodes. Losing one node's instance-store data does not necessarily mean losing the application's data because replicas exist elsewhere.

---

# EBS vs Instance Store

| Feature                         | EBS                                        | Instance Store                       |
| ------------------------------- | ------------------------------------------ | ------------------------------------ |
| Storage location                | Network-attached AWS storage               | Physically local to host             |
| Persistent                      | Yes                                        | No                                   |
| Survives stop                   | Yes                                        | No                                   |
| Survives reboot                 | Yes                                        | Yes                                  |
| Can snapshot with EBS snapshots | Yes                                        | No                                   |
| Can detach and attach elsewhere | Yes, subject to AZ                         | No                                   |
| Very high local IOPS            | Lower than local NVMe in some cases        | Excellent                            |
| Typical use                     | OS, databases, persistent application data | Cache, scratch, temporary processing |

### Exam clue

> "Fastest possible temporary storage."

→ **Instance Store**

> "High IOPS storage and data must persist."

→ **EBS**, usually **io2** when extremely high IOPS are required.

Do not choose Instance Store just because the question says "highest IOPS" if it also says **the data must persist**.

---

# The main storage comparison

| Storage             | Best for                                         |
| ------------------- | ------------------------------------------------ |
| **EBS**             | Persistent block storage attached to EC2         |
| **EFS**             | Shared files across multiple Linux-based clients |
| **Instance Store**  | Very fast temporary local storage                |
| **FSx for Windows** | Windows/SMB/Active Directory file shares         |
| **FSx for Lustre**  | High-performance parallel file systems           |
| **S3**              | Object storage and large-scale durable data      |

---

# Question patterns

> **"Database needs 12,000 IOPS and does not require extreme I/O performance."**

→ **gp3**

---

> **"The application needs more IOPS without increasing storage capacity."**

→ **gp3**

---

> **"The application requires consistent and low-latency performance for an I/O-intensive relational or NoSQL database."**

→ **io1 or io2**

---

> **"Database requires very high, predictable IOPS."**

→ **io1/io2**

---

> **"Application requires 256,000 IOPS and extremely low latency."**

→ **io2 Block Express**

---

> **"Large sequential processing of logs, throughput is the priority."**

→ **st1**

---

> **"Data is rarely accessed and storage cost should be minimized."**

→ **sc1**

---

> **"A legacy application uses Magnetic EBS for a small, infrequently accessed dataset where performance is not important."**

→ **Magnetic (`standard`)**

---

> **"An older question says Magnetic provides the lowest cost per GB and is suitable for infrequently accessed data."**

→ Recognize **Magnetic (`standard`) as a previous-generation EBS volume type**.

Do not confuse it with **sc1**, which is the current Cold HDD option for infrequently accessed, low-cost EBS storage.

---

> **"The company uses gp2 and wants more performance without paying for unnecessary storage."**

→ **Migrate to gp3**

---

> **"Multiple EC2 instances in different AZs must access the same files."**

→ **EFS**

---

> **"Multiple EC2 instances in the same AZ must access the same block volume."**

→ **io1/io2 Multi-Attach**

---

> **"Up to 16 supported EC2 instances need simultaneous read/write access to the same block volume in one AZ."**

→ **io1/io2 Multi-Attach**

---

> **"gp3 Multi-Attach provides multi-AZ resiliency."**

→ **Wrong**

`gp3` does not support Multi-Attach, and Multi-Attach itself is limited to instances in the **same AZ**.

---

> **"A storage option called Spot provides the lowest EBS cost per GB."**

→ **Wrong**

There is no **Spot EBS volume type**.

Spot is an EC2 purchasing model.

---

> **"SR-IOV volume is suitable for boot volumes and small databases."**

→ **Wrong**

SR-IOV is not an EBS volume type.

---

> **"Windows servers need an SMB file share integrated with Active Directory."**

→ **FSx for Windows File Server**

---

> **"Application needs very fast temporary local storage."**

→ **Instance Store**

---

> **"Application needs very fast storage and the data must survive an EC2 stop."**

→ **EBS**

---

> **"The root volume must survive EC2 termination."**

→ **Set `DeleteOnTermination = false`**

---

> **"An EBS volume must be moved to another AZ."**

→ **Snapshot → restore in the destination AZ**

---

> **"A backup must be copied to another AWS Region."**

→ **Copy the EBS snapshot to the destination Region**

---

> **"EBS encryption type."**

→ **KMS by default**

---

> **"All new EBS volumes, including volumes restored from unencrypted snapshots, must automatically be encrypted."**

→ **Enable EBS Encryption by Default for the AWS Region**

---

> **"Automatically create EBS snapshots on a schedule and delete old ones."**

→ **Amazon Data Lifecycle Manager (DLM)**

---

> **"Protect against accidental deletion of EBS snapshots."**

→ **EBS Recycle Bin**

---

> **"A restored EBS volume must have full performance immediately."**

→ **Fast Snapshot Restore**

---

> **"Encrypt an existing unencrypted EBS volume."**

→ **Snapshot → copy with encryption → create encrypted volume**

---

> **"An encrypted EBS volume must protect data both at rest and while moving between EC2 and EBS."**

→ **EBS encryption**

---

> **"A snapshot is created from an encrypted EBS volume."**

→ **The snapshot is automatically encrypted**

---

> **"Create a volume from an encrypted EBS snapshot."**

→ **The new volume is automatically encrypted**

---

> **"Multiple Linux instances in different AZs need access to the same shared file system."**

→ **EFS**

---

> **"Windows + SMB + AD."**

→ **FSx for Windows File Server**

---

> **"Thousands of EFS clients; maximum aggregate performance is more important than latency."**

→ **EFS Max I/O performance mode**

---

> **"Reduce EFS cost for files that are rarely accessed."**

→ **EFS lifecycle management → IA / Archive**

---

> **"A company needs a POSIX-compliant, multi-AZ shared file system for thousands of EC2 instances."**

→ **EFS**

> **"Data can be recreated and the application needs the highest local storage performance."**

→ **Instance Store**

---

# Pocket card

| Keyword                                                           | Answer                                     |
| ----------------------------------------------------------------- | ------------------------------------------ |
| persistent block storage                                          | **EBS**                                    |
| shared file system                                                | **EFS**                                    |
| temporary local storage                                           | **Instance Store**                         |
| random I/O / database                                             | **SSD**                                    |
| large sequential workload / throughput                            | **HDD**                                    |
| general-purpose SSD                                               | **gp3**                                    |
| gp3 independent IOPS/throughput                                   | **Yes**                                    |
| gp3 baseline IOPS                                                 | **3,000**                                  |
| gp3 baseline throughput                                           | **125 MB/s**                               |
| gp3 maximum IOPS                                                  | **80,000**                                 |
| provisioned / predictable IOPS                                    | **io1/io2**                                |
| 256,000 IOPS / extremely high performance                         | **io2 Block Express**                      |
| frequent sequential access                                        | **st1**                                    |
| infrequent / cheapest current HDD storage                         | **sc1**                                    |
| legacy / previous-generation infrequent storage                   | **Magnetic (`standard`)**                  |
| gp2 → better performance flexibility                              | **gp3**                                    |
| same block volume attached to multiple instances                  | **io1/io2 Multi-Attach**                   |
| Multi-Attach limit                                                | **up to 16 instances, same AZ**            |
| gp3 Multi-Attach                                                  | **No**                                     |
| Multi-Attach multi-AZ                                             | **No — same AZ only**                      |
| Elastic Volumes shrink existing EBS volume                        | **No**                                     |
| root volume survives termination                                  | **DeleteOnTermination = false**            |
| new EBS data volume                                               | **format + mount**                         |
| EBS volume usable during snapshot                                 | **Yes — read/write continues**             |
| EBS snapshot does not lock volume                                 | **Volume remains available**               |
| move EBS to another AZ                                            | **snapshot → restore**                     |
| cross-Region EBS DR                                               | **snapshot → copy to Region → restore**    |
| cheaper rarely restored snapshots                                 | **Snapshot Archive**                       |
| recover accidentally deleted snapshots                            | **Recycle Bin**                            |
| immediate full performance after snapshot restore                 | **Fast Snapshot Restore**                  |
| automate EBS snapshot lifecycle                                   | **DLM**                                    |
| EBS Encryption by Default                                         | **Automatically encrypts new EBS volumes** |
| EBS Encryption by Default scope                                   | **Region-level**                           |
| unencrypted snapshot → new volume with default encryption enabled | **Automatically encrypted**                |
| existing unencrypted EBS resources                                | **Not automatically encrypted**            |
| encrypt existing unencrypted EBS                                  | **snapshot → encrypted copy → new volume** |
| EBS data in transit                                               | **encrypted**                              |
| encrypted EBS snapshot                                            | **automatically encrypted**                |
| volume from encrypted snapshot                                    | **automatically encrypted**                |
| shared files across Linux instances                               | **EFS**                                    |
| shared files across AZs                                           | **EFS**                                    |
| Linux NFS file system                                             | **EFS**                                    |
| Windows + SMB + AD                                                | **FSx for Windows File Server**            |
| HPC parallel file system                                          | **FSx for Lustre**                         |
| EFS maximum concurrency                                           | **Max I/O**                                |
| automatic EFS throughput scaling                                  | **Elastic throughput**                     |
| cold EFS files                                                    | **IA / Archive lifecycle**                 |
| fastest temporary EC2 storage                                     | **Instance Store**                         |
| instance store + reboot                                           | **data survives**                          |
| instance store + stop                                             | **data lost**                              |
| instance store + terminate                                        | **data lost**                              |
| instance store + hibernate                                        | **data lost**                              |
| high IOPS + persistence required                                  | **EBS, typically io2 for extreme IOPS**    |
| fake EBS type: Spot                                               | **Not an EBS volume type**                 |
| fake EBS type: SR-IOV                                             | **Not an EBS volume type**                 |
