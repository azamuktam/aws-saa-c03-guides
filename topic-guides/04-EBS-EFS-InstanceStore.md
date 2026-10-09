# Section 4: EBS, EFS & Instance Store

## The idea

* **Amazon Elastic Block Store (EBS)** → persistent block storage attached to EC2.
* **Amazon Elastic File System (EFS)** → shared file storage accessible by multiple Linux-based clients.
* **Instance Store** → very fast, temporary local storage physically attached to the EC2 host.

---

# EBS — Elastic Block Store

Amazon EBS provides persistent block storage for EC2. Typical uses include operating system disks, databases, application files, logs, and persistent data.

## Core EBS rules

### 1. An EBS volume belongs to one Availability Zone (AZ)

An EBS volume cannot be directly attached to an EC2 instance in another AZ.

```text
EBS volume
    ↓
Snapshot
    ↓
Restore in destination AZ
    ↓
New EBS volume
```

For cross-Region migration, copy the snapshot to the destination Region before restoring it.

### 2. EBS is persistent

EBS data survives when an EC2 instance is stopped. Survival after termination depends on `DeleteOnTermination`.

| Volume                | Default setting              | Survives termination? |
| --------------------- | ---------------------------- | --------------------- |
| Root EBS volume       | `DeleteOnTermination = true` | No                    |
| Additional EBS volume | Generally `false`            | Yes                   |

**Exam clue:** The root volume must survive termination → set `DeleteOnTermination = false`.

## EBS volume attachment

Normally, an EBS volume attaches to one EC2 instance at a time.

### EBS Multi-Attach

Supported `io1` and `io2` volumes can attach to **up to 16 supported Nitro-based EC2 instances in the same AZ**. The application must be designed to coordinate shared block storage.

| Feature               | EBS Multi-Attach | EFS         |
| --------------------- | ---------------- | ----------- |
| Shared resource       | Block device     | File system |
| Multi-instance access | Yes              | Yes         |
| Multi-AZ access       | No; same AZ only | Yes         |

Only `io1` and `io2` support Multi-Attach. `gp3`, `st1`, `sc1`, and Magnetic (`standard`) do not.

**Exam clue:** Multiple EC2 instances need the same block volume in one AZ, and the application is cluster-aware → `io1/io2` Multi-Attach.

### Elastic Volumes

Elastic Volumes can increase volume size, change volume type, and adjust supported performance settings. **An existing EBS volume cannot be shrunk.**

**Increase/grow → Yes | Shrink → No**

### Modify EBS volumes and EC2 instance attributes

Do not confuse APIs that modify an EBS volume with APIs that modify the EC2 instance.

| API                        | What it modifies             | Example                                                                                                          |
| -------------------------- | ---------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| `ModifyVolume`             | EBS volume configuration     | Change supported volume type, size, or IOPS; reduce provisioned IOPS on an `io2` volume within its allowed range |
| `ModifyInstanceAttribute`  | EC2 instance attributes      | Change the EBS-optimized (`ebsOptimized`) setting, subject to instance-type support                              |
| CloudWatch `GetMetricData` | Retrieves monitoring metrics | Analyze actual IOPS and throughput usage                                                                         |

**Exam trap:** `ModifyInstanceAttribute` is incorrect when the requirement is to change the provisioned IOPS of an individual EBS volume. Use `ModifyVolume`.

**Can `ModifyVolume` be used while EC2 is running?** Often yes. Supported modifications can be made to an attached, in-use volume without stopping the instance or detaching the volume, subject to applicable limitations. After increasing volume size, extend the filesystem to use the additional capacity. Wait for the previous modification to reach `completed` before starting another modification on the same volume.

---

## Create and mount a new EBS data volume

A newly attached EBS data volume is a block device; Linux does not automatically format or mount it.

```text
Create volume → Attach to EC2 → Format → Mount
                                      ↓
                        Add to /etc/fstab for automatic mounting
```

Example:

```bash
mkfs -t xfs /dev/nvme1n1
mkdir /data
mount /dev/nvme1n1 /data
```

The exact device name depends on the instance and operating system.

**Exam clue:** An attached volume is missing from the expected directory → format and mount it; configure `/etc/fstab` if automatic mounting after reboot is required.

---

# EBS performance: IOPS vs Throughput

* **IOPS (Input/Output Operations Per Second):** Number of read/write operations per second. Important for databases, transactional systems, and small random operations.
* **Throughput:** Amount of data transferred per second, usually MB/s or GB/s. Important for large sequential workloads, log processing, ETL, streaming, and big-data processing.

| Workload requirement          | Think           |
| ----------------------------- | --------------- |
| Many small/random operations  | High IOPS       |
| Large sequential reads/writes | High throughput |

## EBS volume types

### SSD

Used for random I/O, databases, boot volumes, and latency-sensitive workloads.

* `gp3` — general-purpose SSD.
* `io1` / `io2` — provisioned IOPS SSD.

### HDD

Used primarily for large sequential workloads and throughput-oriented processing.

* `st1` — Throughput-Optimized HDD.
* `sc1` — Cold HDD.

**Important:** `st1` and `sc1` cannot be used as root/boot volumes.

### Magnetic (`standard`)

Magnetic is a previous-generation EBS volume type for small datasets accessed infrequently when performance is not the primary concern.

For current EBS choices, `sc1` is the low-cost HDD option for infrequently accessed data.

| Term                  | Meaning                                                   |
| --------------------- | --------------------------------------------------------- |
| Magnetic (`standard`) | Previous-generation EBS; legacy terminology               |
| `sc1`                 | Current Cold HDD for low-cost, infrequently accessed data |

**Exam traps:**

* `Spot` is not an EBS volume type. Spot is an EC2 purchasing option.
* `SR-IOV` is not an EBS volume type. It is an I/O virtualization technology.

---

# General Purpose SSD — gp3

`gp3` is the general-purpose SSD option and a common default choice.

* **3,000 baseline IOPS**
* **125 MB/s baseline throughput**
* IOPS and throughput can be provisioned independently of volume size.
* Supports up to **80,000 IOPS**.

### Example

A workload needs 10,000 IOPS and 100 GB of storage.

With `gp3`, you can configure:

```text
100 GB
10,000 IOPS
```

You do not need to increase storage capacity just to obtain more IOPS.

**Exam clue:** More IOPS without increasing storage capacity → `gp3`.

## gp2 vs gp3

`gp2` ties IOPS to volume size and also supports burst behavior.

* `gp2`: baseline of **3 IOPS per GiB**, subject to volume-size and burst rules.
* `gp3`: baseline of 3,000 IOPS; IOPS and throughput can be configured independently of size.

| Feature                        | gp2            | gp3                       |
| ------------------------------ | -------------- | ------------------------- |
| Baseline IOPS model            | Tied to size   | 3,000 baseline            |
| IOPS independent of size       | No             | Yes                       |
| Throughput independent of size | No             | Yes                       |
| General-purpose SSD choice     | Previous model | Common recommended choice |

**Exam clue:** A company uses `gp2` and needs more performance without paying for unnecessary storage → migrate to `gp3`.

Do not choose `gp3` merely because the question mentions `gp2`; look for independent performance configuration or better price/performance.

---

# Provisioned IOPS SSD — io1 / io2

Use `io1` or `io2` when workloads require high, predictable IOPS and consistently low latency, especially for demanding databases and transactional applications.

Typical workloads:

* High-performance relational databases.
* High-performance NoSQL databases.
* Demanding transactional applications.
* Applications with strict I/O requirements.

| Requirement                                             | Choice        |
| ------------------------------------------------------- | ------------- |
| Normal database or general-purpose workload             | `gp3`         |
| High, predictable IOPS                                  | `io1` / `io2` |
| I/O-intensive database with strict latency requirements | `io1` / `io2` |

**Exam clues:**

* “Consistent, low-latency performance” → Provisioned IOPS.
* “Highly I/O-intensive database” → `io1` / `io2`.

Do not automatically choose Provisioned IOPS just because the workload is a database. Match the choice to the actual performance requirements.

## io2 Block Express

**Amazon EBS io2 Block Express** supports up to **256,000 IOPS** and provides very low latency.

**Exam clues:** “256,000 IOPS” or “sub-millisecond latency” → `io2 Block Express`.

---

# Throughput-Optimized HDD — st1

Use `st1` for large sequential workloads where throughput matters more than random IOPS.

Typical workloads include:

* Log processing.
* Big-data workloads.
* ETL.
* Large sequential datasets.

**Exam clue:** Large sequential reads/writes and high throughput at lower cost → `st1`.

# Cold HDD — sc1

Use `sc1` for infrequently accessed data when minimizing storage cost is the priority.

**Exam clue:** Infrequently accessed data on low-cost current-generation EBS HDD storage → `sc1`.

# Magnetic — standard

Magnetic (`standard`) is a previous-generation EBS volume type for small datasets, infrequent access, and workloads where performance is not the primary concern. Its performance is much lower and less consistent than that of modern SSD types.

**Legacy exam wording:** “Magnetic provides the lowest cost per GB and is suitable for infrequently accessed data.”

Recognize this as **Magnetic (`standard`)**, not `sc1`.

---

# EBS decision guide

| Workload requirement                           | EBS choice            |
| ---------------------------------------------- | --------------------- |
| General-purpose SSD                            | `gp3`                 |
| IOPS independent of volume size                | `gp3`                 |
| High, predictable IOPS                         | `io1` / `io2`         |
| Up to 256,000 IOPS, extreme performance        | `io2 Block Express`   |
| Large sequential workload, throughput priority | `st1`                 |
| Infrequently accessed, low-cost current HDD    | `sc1`                 |
| Legacy Magnetic volume requirement             | Magnetic (`standard`) |

---

# EBS snapshots

An EBS snapshot is a point-in-time backup of an EBS volume. Snapshots are stored in AWS-managed storage and are **incremental after the first snapshot**.

AWS stores the blocks that changed since the previous snapshot rather than making each snapshot a completely independent full copy.

```text
Day 1 → Snapshot A
Day 2 → Snapshot B (changed blocks)
Day 3 → Snapshot C (changed blocks)
```

## Using an EBS volume while a snapshot is in progress

An EBS volume remains available while its snapshot is created. The volume can continue to be read from and written to, and the EC2 instance can continue normal operations.

```text
EC2 → EBS volume
        ├── Read ✅
        ├── Write ✅
        └── Snapshot in progress ✅
```

**Exam trap:** “Can the EBS volume be used while the snapshot is in progress?” → **Yes.** A snapshot does not lock the volume or make it read-only.

## Cross-Region EBS backup

For disaster recovery in another Region:

```text
EBS volume
    ↓
Create snapshot
    ↓
Copy snapshot to destination Region
    ↓
Restore volume there when needed
```

**Exam clue:** Back up EBS data to another AWS Region → create a snapshot and copy it to the destination Region.

## Snapshot Archive

**EBS Snapshot Archive** is intended for snapshots that are rarely accessed. It substantially reduces storage cost but has a slower restore process.

**Exam clue:** Snapshots are rarely restored, and lower backup storage cost matters more than fast recovery → Snapshot Archive.

## Recycle Bin

**Amazon EBS Recycle Bin** protects supported resources, including EBS snapshots, against accidental deletion. Retention rules keep deleted snapshots recoverable for a specified period.

**Exam clue:** Protect accidentally deleted EBS snapshots → Recycle Bin.

## Fast Snapshot Restore

Normally, data restored from an EBS snapshot is loaded on demand, potentially slowing initial reads while blocks are initialized.

**Fast Snapshot Restore (FSR)** pre-initializes the restored volume so that it can provide full performance immediately.

Trade-off: Faster recovery and immediate performance, but additional cost.

**Exam clue:** A restored EBS volume must provide full performance immediately, without the initial read penalty → Fast Snapshot Restore.

## Data Lifecycle Manager — DLM

**Amazon Data Lifecycle Manager (DLM)** automates EBS snapshot lifecycle operations, including creation, retention, deletion, and selected cross-Region snapshot-copy workflows.

Example:

```text
Create a snapshot daily
        ↓
Keep the last 7
        ↓
Delete older snapshots
```

**Exam clue:** Automatically create, retain, and delete EBS snapshots on a schedule → Amazon DLM.

For broader centralized backup management across multiple AWS services, consider **AWS Backup**.

---

# EBS encryption — AWS KMS

EBS encryption protects data at rest, data in transit between EC2 and EBS, and snapshots.

| Scenario                                                               | Result                    |
| ---------------------------------------------------------------------- | ------------------------- |
| Encrypted EBS volume                                                   | Data at rest is encrypted |
| Data moving between EC2 and EBS                                        | Encrypted                 |
| Snapshot of an encrypted volume                                        | Automatically encrypted   |
| Volume created from an encrypted snapshot                              | Automatically encrypted   |
| Claim that only data in the volume is encrypted                        | Incorrect                 |
| Claim that a snapshot of an encrypted volume is unencrypted            | Incorrect                 |
| Claim that a volume restored from an encrypted snapshot is unencrypted | Incorrect                 |

## EBS Encryption by Default

**EBS Encryption by Default** is a **Region-level account setting** that automatically encrypts new EBS volumes in that Region.

A new EBS volume restored from an unencrypted snapshot is also automatically encrypted when the setting applies.

```text
Enable EBS Encryption by Default
               ↓
          AWS Region
               ↓
       New EBS volumes
               ↓
      Automatically encrypted
```

It does not automatically encrypt existing unencrypted EBS volumes or snapshots.

## Encrypt an existing unencrypted EBS volume

You cannot simply toggle an existing unencrypted volume to encrypted in place. Use:

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

**Exam clue:** Encrypt an existing unencrypted EBS volume → snapshot → copy with encryption → create a new encrypted volume.

---

# EFS — Elastic File System

Amazon EFS is a managed, scalable, POSIX-compliant **file system** that primarily serves Linux-based workloads through NFSv4.

It grows and shrinks automatically as files are added or removed, and supports simultaneous access by multiple clients.

```text
             EFS
              │
      ┌───────┼───────┐
      ↓       ↓       ↓
     EC2     EC2     EC2
```

All clients can access the same shared files.

## EFS vs EBS

| Feature                  | EBS                             | EFS                                        |
| ------------------------ | ------------------------------- | ------------------------------------------ |
| Storage type             | Block                           | File                                       |
| Shared by many instances | Normally no                     | Yes                                        |
| Multi-AZ                 | Volume is AZ-specific           | Yes                                        |
| Access model             | Block device                    | NFS file system                            |
| Typical uses             | OS, databases, application disk | Shared files                               |
| Capacity management      | Provision volume size           | Grows/shrinks automatically                |
| Typical clients          | EC2                             | Multiple EC2 instances and compute clients |

## EFS use cases

* User uploads.
* Shared application files.
* Content management systems.
* WordPress content.
* Shared configuration and data across Linux instances.
* Applications behind an Auto Scaling Group (ASG) that require shared files.

**Exam clue:** Multiple EC2 instances in different AZs must access the same files → EFS.

## EFS and Windows

EFS is not the standard choice for native Windows file-sharing requirements involving SMB or Active Directory.

**Amazon FSx for Windows File Server** is the appropriate choice for Windows file shares requiring:

* Server Message Block (SMB).
* Windows file-sharing features.
* Active Directory integration.

**Exam clue:** Windows + SMB + Active Directory → FSx for Windows File Server.

## EFS storage and cost

* EFS automatically grows and shrinks as data changes; you do not pre-provision a fixed capacity as with EBS.
* EFS generally costs more per GB than EBS, but does not charge for unused provisioned capacity in the same way.
* EFS lifecycle management can move less frequently accessed files into lower-cost storage classes.

## EFS performance modes

| Mode            | Behavior                                                                                       |
| --------------- | ---------------------------------------------------------------------------------------------- |
| General Purpose | Default and recommended for most workloads; prioritizes lower latency                          |
| Max I/O         | Designed for very high concurrency and aggregate throughput where higher latency is acceptable |

**Exam clue:** Thousands of concurrent clients, maximum aggregate throughput more important than latency → Max I/O.

For most applications, choose General Purpose.

## EFS throughput modes

| Mode        | Behavior                                                                                         |
| ----------- | ------------------------------------------------------------------------------------------------ |
| Elastic     | Automatically scales throughput with workload demand; a modern default choice for many workloads |
| Provisioned | Explicitly configures throughput independently of stored data size                               |
| Bursting    | Throughput depends on stored data and the EFS bursting model                                     |

**Exam clue:** Automatically adapts throughput to workload demand → Elastic throughput.

## EFS lifecycle management

EFS can move less frequently accessed files into lower-cost storage classes.

```text
Frequently accessed → EFS Standard
Less frequently accessed → EFS IA
Very cold data → EFS Archive
```

**Exam clue:** Reduce costs for files that have not been accessed for a long time → EFS lifecycle management.

---

# Instance Store

Instance Store provides local storage physically attached to the EC2 host.

* Very low latency.
* Potentially extremely high I/O performance.
* No network hop like network-attached EBS.
* **Ephemeral:** data must be reproducible, disposable, or recoverable elsewhere.

## Instance Store data persistence

Instance Store data survives an EC2 **reboot**, but not a stop, hibernate, or termination.

| EC2 action | Instance Store data |
| ---------- | ------------------- |
| Reboot     | Survives            |
| Stop       | Lost                |
| Hibernate  | Lost                |
| Terminate  | Lost                |

**Exam clue:** Temporary cache, scratch space, or data replicated elsewhere → Instance Store.

## When Instance Store is appropriate

* Temporary files.
* Scratch space.
* Caching.
* Intermediate processing data.
* Data replicated by the application.
* High-speed local processing.

For example, a Cassandra cluster can replicate data across multiple nodes. Losing one node's Instance Store data does not necessarily lose the application's data because other replicas exist.

## EBS vs Instance Store

| Feature                         | EBS                            | Instance Store                       |
| ------------------------------- | ------------------------------ | ------------------------------------ |
| Location                        | Network-attached AWS storage   | Physically local to host             |
| Persistent                      | Yes                            | No                                   |
| Survives stop                   | Yes                            | No                                   |
| Survives reboot                 | Yes                            | Yes                                  |
| EBS snapshots supported         | Yes                            | No                                   |
| Can detach and attach elsewhere | Yes, subject to AZ             | No                                   |
| Very high local IOPS            | Depends on volume and instance | Excellent                            |
| Typical use                     | OS, databases, persistent data | Cache, scratch, temporary processing |

**Exam clues:**

* “Fastest possible temporary storage” → Instance Store.
* “High IOPS and data must persist” → EBS, typically `io2` for extreme IOPS.

Do not choose Instance Store solely for its performance if the data must survive an EC2 stop.

---

# Main storage comparison

| Storage                         | Best for                                         |
| ------------------------------- | ------------------------------------------------ |
| **EBS**                         | Persistent block storage attached to EC2         |
| **EFS**                         | Shared files across multiple Linux-based clients |
| **Instance Store**              | Very fast temporary local storage                |
| **FSx for Windows File Server** | Windows/SMB/Active Directory file shares         |
| **FSx for Lustre**              | High-performance parallel file systems           |
| **Amazon S3**                   | Object storage and large-scale durable data      |

---

# Question patterns

> **“Database needs 12,000 IOPS and does not require extreme I/O performance.”**

→ `gp3`, provided its configured performance meets the requirements.

> **“The application needs more IOPS without increasing storage capacity.”**

→ `gp3`.

> **“The application needs consistent, low-latency performance for an I/O-intensive relational or NoSQL database.”**

→ `io1` or `io2`.

> **“Database requires very high, predictable IOPS.”**

→ `io1/io2`.

> **“Application requires 256,000 IOPS and extremely low latency.”**

→ `io2 Block Express`.

> **“Large sequential processing of logs; throughput is the priority.”**

→ `st1`.

> **“Data is rarely accessed and storage cost should be minimized.”**

→ `sc1`.

> **“A legacy application uses Magnetic EBS for a small, infrequently accessed dataset where performance is not important.”**

→ Magnetic (`standard`).

> **“An older question says Magnetic provides the lowest cost per GB for infrequently accessed data.”**

→ Recognize Magnetic (`standard`) as a previous-generation EBS volume type. Do not confuse it with `sc1`, the current Cold HDD option.

> **“The company uses gp2 and wants more performance without paying for unnecessary storage.”**

→ Migrate to `gp3`.

> **“Multiple EC2 instances in different AZs need access to the same files.”**

→ EFS.

> **“Multiple EC2 instances in the same AZ need access to the same block volume.”**

→ `io1/io2` Multi-Attach.

> **“Up to 16 supported EC2 instances need simultaneous read/write access to the same block volume in one AZ.”**

→ `io1/io2` Multi-Attach.

> **“gp3 Multi-Attach provides multi-AZ resiliency.”**

→ Incorrect. `gp3` does not support Multi-Attach, and Multi-Attach is limited to the same AZ.

> **“A storage option called Spot provides the lowest EBS cost per GB.”**

→ Incorrect. Spot is an EC2 purchasing option, not an EBS volume type.

> **“SR-IOV volume is suitable for boot volumes and small databases.”**

→ Incorrect. SR-IOV is not an EBS volume type.

> **“Windows servers need an SMB file share integrated with Active Directory.”**

→ FSx for Windows File Server.

> **“The application needs very fast temporary local storage.”**

→ Instance Store.

> **“The application needs fast storage, and data must survive an EC2 stop.”**

→ EBS.

> **“The root volume must survive EC2 termination.”**

→ Set `DeleteOnTermination = false`.

> **“An EBS volume must be moved to another AZ.”**

→ Snapshot → restore in the destination AZ.

> **“An EBS backup must be copied to another AWS Region.”**

→ Copy the EBS snapshot to the destination Region.

> **“EBS volume configuration needs to change, including provisioned IOPS.”**

→ Use `ModifyVolume`, not `ModifyInstanceAttribute`.

> **“The EC2 instance needs its EBS-optimized attribute changed.”**

→ `ModifyInstanceAttribute`, subject to instance-type support.

> **“Determine actual IOPS and throughput usage from CloudWatch metrics.”**

→ CloudWatch `GetMetricData`.

> **“All new EBS volumes, including volumes restored from unencrypted snapshots, must automatically be encrypted.”**

→ Enable EBS Encryption by Default for the Region.

> **“Encrypt an existing unencrypted EBS volume.”**

→ Snapshot → copy with encryption → create a new encrypted volume.

> **“An encrypted EBS volume must protect data at rest and while moving between EC2 and EBS.”**

→ EBS encryption.

> **“A snapshot is created from an encrypted EBS volume.”**

→ The snapshot is automatically encrypted.

> **“Create a volume from an encrypted EBS snapshot.”**

→ The new volume is automatically encrypted.

> **“Automatically create and retain EBS snapshots on a schedule.”**

→ Amazon Data Lifecycle Manager (DLM).

> **“Protect accidentally deleted EBS snapshots.”**

→ EBS Recycle Bin.

> **“A restored EBS volume must provide full performance immediately.”**

→ Fast Snapshot Restore.

> **“Multiple Linux instances in different AZs need access to the same shared file system.”**

→ EFS.

> **“Linux instances need a POSIX-compliant shared file system across AZs.”**

→ EFS.

> **“Windows + SMB + Active Directory.”**

→ FSx for Windows File Server.

> **“Thousands of EFS clients; maximum aggregate performance matters more than latency.”**

→ EFS Max I/O performance mode.

> **“Reduce EFS cost for files that are rarely accessed.”**

→ EFS lifecycle management → IA / Archive.

> **“Data can be recreated, and the application needs the highest local storage performance.”**

→ Instance Store.

---

# Pocket card

| Keyword                                               | Answer                                                    |
| ----------------------------------------------------- | --------------------------------------------------------- |
| Persistent block storage                              | **EBS**                                                   |
| Shared file system                                    | **EFS**                                                   |
| Temporary local storage                               | **Instance Store**                                        |
| Random I/O / database                                 | **SSD**                                                   |
| Large sequential workload / throughput                | **HDD**                                                   |
| General-purpose SSD                                   | **gp3**                                                   |
| gp3 baseline IOPS                                     | **3,000**                                                 |
| gp3 baseline throughput                               | **125 MB/s**                                              |
| gp3 maximum IOPS                                      | **80,000**                                                |
| gp3 IOPS/throughput independent of size               | **Yes**                                                   |
| Provisioned, predictable IOPS                         | **io1/io2**                                               |
| Extreme performance / 256,000 IOPS                    | **io2 Block Express**                                     |
| Frequent sequential processing                        | **st1**                                                   |
| Infrequent-access, low-cost current HDD               | **sc1**                                                   |
| Legacy, previous-generation infrequent-access storage | **Magnetic (`standard`)**                                 |
| gp2 → more flexible performance                       | **gp3**                                                   |
| Same block volume attached to multiple instances      | **io1/io2 Multi-Attach**                                  |
| Multi-Attach limit                                    | **Up to 16 supported Nitro-based instances, same AZ**     |
| gp3 Multi-Attach                                      | **No**                                                    |
| Multi-Attach multi-AZ                                 | **No — same AZ only**                                     |
| Shrink an existing EBS volume                         | **Not supported**                                         |
| Modify individual volume IOPS                         | **`ModifyVolume`**                                        |
| Modify EC2 instance attributes                        | **`ModifyInstanceAttribute`**                             |
| Retrieve CloudWatch metrics                           | **`GetMetricData`**                                       |
| Change supported EBS configuration while in use       | **Often possible with `ModifyVolume`; check limitations** |
| Root volume survives termination                      | **`DeleteOnTermination = false`**                         |
| New EBS data volume                                   | **Format + mount**                                        |
| Volume usable during snapshot                         | **Yes — reads/writes continue**                           |
| Move EBS to another AZ                                | **Snapshot → restore**                                    |
| Cross-Region EBS disaster recovery                    | **Snapshot → copy to Region → restore**                   |
| Cheaper rarely restored snapshots                     | **Snapshot Archive**                                      |
| Recover accidentally deleted snapshots                | **Recycle Bin**                                           |
| Immediate performance after snapshot restore          | **Fast Snapshot Restore**                                 |
| Automate EBS snapshot lifecycle                       | **DLM**                                                   |
| EBS Encryption by Default                             | **Automatically encrypts new EBS volumes**                |
| EBS Encryption by Default scope                       | **Region-level**                                          |
| Existing unencrypted EBS resources                    | **Not automatically encrypted**                           |
| Encrypt existing unencrypted EBS volume               | **Snapshot → encrypted copy → new volume**                |
| Encrypted EBS data in transit                         | **Yes**                                                   |
| Snapshot of encrypted volume                          | **Automatically encrypted**                               |
| Volume created from encrypted snapshot                | **Automatically encrypted**                               |
| Shared files across Linux instances/AZs               | **EFS**                                                   |
| Linux NFS / POSIX file system                         | **EFS**                                                   |
| Windows + SMB + Active Directory                      | **FSx for Windows File Server**                           |
| HPC parallel file system                              | **FSx for Lustre**                                        |
| EFS maximum-concurrency performance mode              | **Max I/O**                                               |
| Automatic EFS throughput scaling                      | **Elastic throughput**                                    |
| Cold EFS files                                        | **IA / Archive lifecycle**                                |
| Fastest temporary EC2 storage                         | **Instance Store**                                        |
| Instance Store + reboot                               | **Data survives**                                         |
| Instance Store + stop/hibernate/terminate             | **Data lost**                                             |
| High IOPS + persistence                               | **EBS; typically io2 for extreme IOPS**                   |
| Fake EBS type: Spot                                   | **Not an EBS volume type**                                |
| Fake EBS type: SR-IOV                                 | **Not an EBS volume type**                                |
