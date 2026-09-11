# Section 25: Storage Gateway, DataSync & Transfer Family

## Big picture

These services solve **different problems**:

* **Storage Gateway** → on-premises systems **access AWS storage**
* **DataSync** → **move/copy data** between storage systems
* **Transfer Family** → people or companies **upload/download files** using SFTP/FTP/FTPS/AS2

The easiest memory trick:

> **Gateway = ACCESS**
> **DataSync = COPY**
> **Transfer Family = FILE TRANSFER**

---

# 1. Storage Gateway

Storage Gateway connects your **on-premises environment** to AWS storage.

```text
On-premises
   ↓
Storage Gateway
   ↓
AWS
```

The important point:

> Your existing on-premises applications can keep using familiar storage protocols.

Storage Gateway has several types.

---

## S3 File Gateway

Your application uses a normal **file share** using:

* NFS
* SMB

But the actual files are stored in **Amazon S3**.

```text
Application
    ↓
NFS / SMB
    ↓
S3 File Gateway
    ↓
S3
```

### Use it when

> "We have applications that need normal file access, but we want the files stored in S3."

### Exam keywords

* NFS or SMB
* files
* S3
* on-premises file share
* replace NAS

### Remember

**S3 File Gateway = file access to S3**

---

## FSx File Gateway

This is similar to S3 File Gateway, but the backend is:

**Amazon FSx for Windows File Server**

It provides on-premises access to FSx using **SMB**.

```text
On-premises
    ↓
SMB
    ↓
FSx File Gateway
    ↓
FSx for Windows
```

### Use it when

> "On-premises users need low-latency access to an FSx for Windows file share."

### Remember

**FSx File Gateway = SMB + FSx for Windows**

---

# Volume Gateway

Volume Gateway provides **block storage** to on-premises applications using:

**iSCSI**

There are two types:

* Cached
* Stored

The important question is:

> **Where is the main/full dataset?**

---

## Volume Gateway — Cached

The **main data is in AWS/S3**.

Only frequently used data is kept locally for fast access.

```text
S3
= main/full dataset

Local
= frequently used data
```

### Use it when

> "Our on-premises storage is running out of space and we want to expand capacity using AWS."

### Remember

**Cached = cloud is primary**

---

## Volume Gateway — Stored

The **main/full dataset stays on-premises**.

AWS S3 stores **snapshots** for backup and disaster recovery.

```text
Local
= full dataset

S3
= snapshots / backup
```

### Use it when

> "The entire dataset must be available locally with low latency, but we also want AWS backup."

### Remember

**Stored = local is primary**

---

## Cached vs Stored

|              | Cached               | Stored                     |
| ------------ | -------------------- | -------------------------- |
| Main dataset | **S3**               | **On-premises**            |
| Local data   | Frequently used data | **Entire dataset**         |
| Main use     | Increase capacity    | Fast local access + backup |
| AWS role     | Primary storage      | Backup/snapshots           |

### Important exam trap

> "Low-latency access to the **entire dataset**"

→ **Volume Gateway — Stored**

Why?

Because with Cached, only frequently used data is local.

---

# Tape Gateway

Tape Gateway is for companies that use **physical tape backups**.

It provides a **Virtual Tape Library (VTL)** so existing backup software can continue working.

The data is stored in AWS instead of physical tapes.

```text
Backup software
      ↓
Tape Gateway
      ↓
S3 / Glacier
```

### Use it when

> "Replace physical tape infrastructure but keep the existing backup software."

### Exam keyword

**Tape → Tape Gateway**

---

# 2. DataSync

## The main idea

**DataSync = move/copy data**

It is used for:

* migrations
* scheduled transfers
* synchronization
* large data transfers

Examples:

```text
On-premises NFS → S3
On-premises SMB → EFS

S3 → EFS
EFS → FSx
```

For on-premises storage, DataSync commonly uses an **agent**.

DataSync can also preserve things such as:

* file metadata
* permissions

It supports **bandwidth throttling** so you can control how much network bandwidth the transfer uses.

---

## Example

> "Copy 50 TB from an on-premises NAS to S3 while preserving file metadata."

→ **DataSync**

Why?

Because the requirement is to **move the data**.

---

## DataSync vs Storage Gateway

This is one of the most important differences.

### DataSync

> **I need to copy/move the data.**

```text
Old storage
    ↓
DataSync
    ↓
New storage
```

### Storage Gateway

> **I still need my on-premises applications to access AWS storage.**

```text
On-premises application
        ↓
Storage Gateway
        ↓
AWS storage
```

### Easy rule

> **Move the data → DataSync**
> **Keep using the storage → Storage Gateway**

---

# 3. Transfer Family

Transfer Family provides a **managed file-transfer server**.

Supported protocols include:

* SFTP
* FTPS
* FTP
* AS2

The files can be stored in:

* S3
* EFS

---

## Example

A company has business partners that already upload files using SFTP.

They don't want to change their existing process.

```text
Partner
   ↓
SFTP
   ↓
Transfer Family
   ↓
S3 / EFS
```

Answer:

**AWS Transfer Family**

You do **not** need to build and manage your own SFTP server on EC2.

---

# Storage Gateway vs DataSync vs Transfer Family

| Service             | Main purpose                                  | Example                            |
| ------------------- | --------------------------------------------- | ---------------------------------- |
| **Storage Gateway** | Access AWS storage from on-premises           | On-prem app accesses files in S3   |
| **DataSync**        | Move/copy data                                | Migrate 50 TB from NAS to S3       |
| **Transfer Family** | File upload/download using transfer protocols | Partner uploads files through SFTP |

---

# Storage Gateway types

| Requirement                                     | Answer                      |
| ----------------------------------------------- | --------------------------- |
| Files in S3, accessed using NFS/SMB             | **S3 File Gateway**         |
| SMB access to FSx for Windows                   | **FSx File Gateway**        |
| Increase on-premises capacity, cloud is primary | **Volume Gateway – Cached** |
| Entire dataset local + AWS backup               | **Volume Gateway – Stored** |
| Replace physical tape backups                   | **Tape Gateway**            |

---

# Common Exam Questions

### "Replace physical tape backups"

→ **Tape Gateway**

---

### "Applications use NFS/SMB but files should be stored in S3"

→ **S3 File Gateway**

---

### "On-premises applications need SMB access to FSx for Windows"

→ **FSx File Gateway**

---

### "On-premises storage is running out of space"

→ **Volume Gateway – Cached**

Because the main data is stored in AWS.

---

### "Entire dataset must remain local for low-latency access, but backups should go to AWS"

→ **Volume Gateway – Stored**

---

### "Migrate 50 TB from an on-premises NAS to S3"

→ **DataSync**

---

### "Copy files every night from an on-premises SMB server to EFS"

→ **DataSync**

Scheduled transfer = DataSync.

---

### "Transfer data between S3 and EFS"

→ **DataSync**

DataSync can also move data between AWS storage services.

---

### "Business partners upload files using SFTP"

→ **Transfer Family**

---

# Final memory card

```text
Storage Gateway
= ACCESS AWS storage from on-premises

DataSync
= MOVE / COPY data

Transfer Family
= SFTP / FTPS / FTP / AS2
  for file transfer
```

### Storage Gateway

```text
S3 files       → S3 File Gateway
FSx Windows    → FSx File Gateway
Cloud primary  → Volume Gateway Cached
Local primary  → Volume Gateway Stored
Tape backups   → Tape Gateway
```

### The most important distinction

```text
"I need to MOVE the data"
        ↓
     DataSync

"I need to KEEP USING the storage"
        ↓
  Storage Gateway

"Someone needs to UPLOAD FILES using SFTP"
        ↓
  Transfer Family
```

### One more service to remember

If the question says:

> **No network / extremely large data / offline transfer**

→ **Snow Family**
