# Section 25: Storage Gateway, DataSync & Transfer Family

## Big picture

These services solve different problems:

* **Storage Gateway** → on-premises **access to AWS storage**
* **DataSync** → **move/copy data**
* **Transfer Family** → **file transfers** using SFTP/FTP/FTPS/AS2

```text
Gateway         = ACCESS
DataSync        = COPY
Transfer Family = FILE TRANSFER
```

---

# 1. Storage Gateway

Connects on-premises systems to AWS storage while allowing existing applications to use familiar storage protocols.

```text
On-premises
   ↓
Storage Gateway
   ↓
AWS storage
```

## S3 File Gateway

Provides **NFS/SMB** file access while storing files in **S3**.

```text
Application → NFS/SMB → S3 File Gateway → S3
```

### Use when

* Existing file-based applications need S3
* On-premises file share / NAS replacement
* NFS or SMB + S3

> **NFS/SMB + S3 → S3 File Gateway**

---

## FSx File Gateway

Provides **SMB** access from on-premises to **FSx for Windows File Server**.

```text
On-premises → SMB → FSx File Gateway → FSx for Windows
```

> **SMB + FSx for Windows → FSx File Gateway**

---

# Volume Gateway

Provides **block storage via iSCSI** not NFS or SMB to on-premises applications.

Two types:

* Cached
* Stored

The key question:

> **Where is the main/full dataset?**

## Volume Gateway — Cached

**Main data = S3/AWS**

Only frequently accessed data is cached locally.

```text
S3
= main dataset

Local
= frequently used data
```

> On-premises storage running out of space / AWS becomes primary → **Cached**

## Volume Gateway — Stored

**Main/full dataset = on-premises**

S3 stores snapshots for backup/DR.

```text
Local
= full dataset

S3
= snapshots / backup
```

> Entire dataset must stay local + AWS backup → **Stored**

### Cached vs Stored

|              | **Cached**           | **Stored**                        |
| ------------ | -------------------- | --------------------------------- |
| Main dataset | **S3**               | **On-premises**                   |
| Local        | Frequently used data | **Entire dataset**                |
| Main use     | Increase capacity    | Low-latency local access + backup |
| AWS role     | Primary storage      | Backup/snapshots                  |

**Trap:**

> Low-latency access to the **entire dataset** → **Volume Gateway Stored**

---

# Tape Gateway

Replaces physical tape infrastructure with **virtual tapes in AWS**.

Provides a **Virtual Tape Library (VTL)** so existing backup software can continue working.

```text
Backup software
      ↓
Tape Gateway
      ↓
S3 / Glacier
```

> Physical tape replacement + existing backup software → **Tape Gateway**

**Keyword:** `Tape → Tape Gateway`

---

# 2. DataSync

**DataSync = move/copy data.**

Use for:

* Migrations
* Scheduled transfers
* Synchronization
* Large data transfers

Examples:

```text
On-prem NFS → S3
On-prem SMB → EFS
S3 → EFS
EFS → FSx
```

For on-premises storage, DataSync commonly uses an **agent**.

It can preserve:

* File metadata
* Permissions

It supports **bandwidth throttling**.

### Example

> Copy 50 TB from on-premises NAS to S3 while preserving metadata → **DataSync**

---

# DataSync vs Storage Gateway

| Requirement                                 | Answer              |
| ------------------------------------------- | ------------------- |
| **Move/copy the data**                      | **DataSync**        |
| **Keep using AWS storage from on-premises** | **Storage Gateway** |

```text
Old storage
   ↓
DataSync
   ↓
New storage
```

vs.

```text
On-prem application
       ↓
Storage Gateway
       ↓
AWS storage
```

> **Move data → DataSync**
> **Access AWS storage → Storage Gateway**

---

# 3. Transfer Family

Provides a **managed file-transfer server**.

Supported protocols:

* **SFTP**
* **FTPS**
* **FTP**
* **AS2**

Storage backends:

* **S3**
* **EFS**

Example:

```text
Partner
   ↓
SFTP
   ↓
Transfer Family
   ↓
S3 / EFS
```

> Business partners already use SFTP → **Transfer Family**

No need to build/manage an SFTP server on EC2.

### Small exam note

> **SFTP does not require EFS → Transfer Family supports S3 or EFS**

> **SFTP + only approved public IPs → VPC-hosted internet-facing endpoint + Security Group**

```text
Approved IP
    ↓
VPC endpoint + Security Group
    ↓
Transfer Family
    ↓
S3 / EFS
```

> Normal **public Transfer Family endpoint + source-IP allow list** → ❌

---

# Storage Gateway vs DataSync vs Transfer Family

| Service             | Main purpose                  | Example                       |
| ------------------- | ----------------------------- | ----------------------------- |
| **Storage Gateway** | On-prem access to AWS storage | On-prem app accesses S3 files |
| **DataSync**        | Move/copy data                | Migrate 50 TB NAS → S3        |
| **Transfer Family** | File transfer protocols       | Partner uploads through SFTP  |

---

# Storage Gateway types

| Requirement                                | Answer                      |
| ------------------------------------------ | --------------------------- |
| Files in S3 via NFS/SMB                    | **S3 File Gateway**         |
| SMB access to FSx for Windows              | **FSx File Gateway**        |
| Cloud is primary / increase local capacity | **Volume Gateway – Cached** |
| Entire dataset local + AWS backup          | **Volume Gateway – Stored** |
| Replace physical tape backups              | **Tape Gateway**            |

---

# Common Exam Questions

> **"Replace physical tape backups."**
> → **Tape Gateway**

> **"Applications use NFS/SMB but files should be stored in S3."**
> → **S3 File Gateway**

> **"On-premises applications need SMB access to FSx for Windows."**
> → **FSx File Gateway**

> **"On-premises storage is running out of space."**
> → **Volume Gateway – Cached**

> **"Entire dataset must remain local for low-latency access, but backups should go to AWS."**
> → **Volume Gateway – Stored**

> **"Copy 50 TB from on-premises NAS to S3 while preserving metadata."**
> → **DataSync**

> **"Copy files every night from an on-premises SMB server to EFS."**
> → **DataSync**

> **"Transfer data between S3 and EFS."**
> → **DataSync**

> **"Business partners upload files using SFTP."**
> → **Transfer Family**

> **"SFTP service does not need a managed EC2 server."**
> → **Transfer Family**

> **"Transfer Family can use S3 as the storage backend."**
> → **Transfer Family + S3**

> **"Transfer Family can use EFS as the storage backend."**
> → **Transfer Family + EFS**

> **"SFTP service must accept connections only from approved public IP addresses."**
> → **Transfer Family + VPC-hosted internet-facing endpoint + Security Group**

> **"Use a normal public Transfer Family endpoint and restrict clients by source IP."**
> → **Wrong**; use a VPC-hosted internet-facing endpoint with a Security Group.

---

# Pocket card

| Keyword                         | Answer                            |
| ------------------------------- | --------------------------------- |
| On-prem → AWS storage access    | **Storage Gateway**               |
| Move/copy/sync data             | **DataSync**                      |
| SFTP/FTPS/FTP/AS2               | **Transfer Family**               |
| NFS/SMB → S3                    | **S3 File Gateway**               |
| SMB → FSx Windows               | **FSx File Gateway**              |
| Cloud primary                   | **Volume Gateway Cached**         |
| Local primary + AWS backup      | **Volume Gateway Stored**         |
| Physical tape replacement       | **Tape Gateway**                  |
| DataSync on-prem                | **Agent**                         |
| DataSync bandwidth control      | **Bandwidth throttling**          |
| Transfer Family storage         | **S3 / EFS**                      |
| SFTP + trusted public IPs       | **VPC endpoint + Security Group** |
| Public endpoint + IP allow list | **Not the intended solution**     |
| SFTP requires EFS?              | **No, S3 also works**             |

### One important extra service

> **No network / offline transfer / extremely large data**
> → **Snow Family**
