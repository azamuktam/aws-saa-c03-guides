# Section 26: FSx Family

## The idea

**Amazon FSx provides managed file systems for specific use cases and file-system technologies.**

There are four important FSx types:

* **FSx for Windows File Server** → Windows workloads
* **FSx for Lustre** → high-performance computing and S3-based workloads
* **FSx for NetApp ONTAP** → NetApp workloads and multi-protocol access
* **FSx for OpenZFS** → ZFS workloads and Linux/NFS workloads

The easiest way to choose:

> **Look for the technology or workload in the question.**

```text
Windows / SMB / AD
→ FSx for Windows

HPC / ML / S3 high-performance processing
→ FSx for Lustre

NetApp / SnapMirror / NFS + SMB + iSCSI
→ FSx for NetApp ONTAP

ZFS
→ FSx for OpenZFS
```

### Key vocabulary

* **SMB** = Windows file-sharing protocol
* **NFS** = common Linux/Unix file-sharing protocol
* **HPC** = High-Performance Computing
* **NTFS** = Windows file system

---

## The four specialists

| FSx type                        | Main keywords                              | Main use                          |
| ------------------------------- | ------------------------------------------ | --------------------------------- |
| **FSx for Windows File Server** | Windows, SMB, NTFS, Active Directory, .NET | Windows file shares               |
| **FSx for Lustre**              | HPC, ML, rendering, high throughput, S3    | High-performance workloads        |
| **FSx for NetApp ONTAP**        | NetApp, SnapMirror, NFS, SMB, iSCSI        | NetApp and multi-protocol storage |
| **FSx for OpenZFS**             | ZFS, NFS, Linux file server migration      | ZFS/Linux workloads               |

---

## FSx for Windows File Server

This is a **managed Windows file system**.

It uses:

* **SMB**
* **NTFS**
* **Active Directory**

Applications and users can access it as a Windows file share.

```text
Windows application
       ↓
      SMB
       ↓
FSx for Windows
```

It can integrate with:

* AWS Managed Microsoft AD
* On-premises Active Directory

It supports **Multi-AZ deployment** for high availability.

### Use it when

> "Windows applications need shared storage."

or:

> "The application uses SMB and Active Directory."

### Important exam distinction

**EFS is designed for Linux/NFS workloads.**

If the question specifically requires:

* Windows
* SMB
* Windows file shares
* Active Directory

→ **FSx for Windows File Server**

### Remember

> **Windows → FSx for Windows**

---

## FSx for Lustre

FSx for Lustre is designed for **very high-performance workloads**.

Typical workloads:

* HPC
* machine learning training
* video rendering
* financial simulations
* genomics
* other workloads requiring very high throughput

It supports **very high throughput and large numbers of IOPS**.

### Important feature: S3 integration

FSx for Lustre can work directly with data stored in **S3**.

```text
S3
 ↓
FSx for Lustre
 ↓
HPC / ML workload
```

This is especially useful when the data is already stored in S3 but the application needs a **high-performance file system** to process it.

### Use it when

> "An ML training job needs very fast access to a dataset stored in S3."

→ **FSx for Lustre**

### Remember

> **HPC + high performance + S3 → Lustre**

---

## FSx for Lustre deployment types

There are two important deployment types:

| Type           | Main use                           | Data protection                  |
| -------------- | ---------------------------------- | -------------------------------- |
| **Scratch**    | Temporary or short-term processing | No replication                   |
| **Persistent** | Longer-term workloads              | Data is replicated within the AZ |

### Scratch

Use Scratch when:

* the data is temporary
* the workload is short-term
* maximum performance is important
* lower cost is preferred

If the file system fails, data on the scratch file system can be lost.

### Persistent

Use Persistent when:

* the workload lasts longer
* the data needs more protection
* the file system is not simply temporary processing storage

### Remember

> **Temporary → Scratch**
> **Longer-term → Persistent**

---

## FSx for NetApp ONTAP

FSx for NetApp ONTAP is a **managed NetApp file system**.

It is especially useful when a company already uses **NetApp** on-premises and wants to move to AWS without changing everything.

It supports:

* **NFS**
* **SMB**
* **iSCSI**

This makes it a **multi-protocol** file system.

### Important feature: SnapMirror

NetApp **SnapMirror** can replicate data between NetApp systems.

This makes FSx for NetApp ONTAP useful for:

* migration
* replication
* disaster recovery

### Example

> "The company uses NetApp on-premises and wants to migrate to AWS with minimal changes."

→ **FSx for NetApp ONTAP**

### Important exam distinction

If the question says:

> "Linux and Windows clients both need access to the same file system."

Then:

* Linux → NFS
* Windows → SMB

→ **FSx for NetApp ONTAP**

### Remember

> **NetApp → ONTAP**
> **NFS + SMB + iSCSI → ONTAP**

---

## FSx for OpenZFS

FSx for OpenZFS is a **managed OpenZFS file system**.

It is useful for workloads using:

* ZFS
* NFS
* Linux/Unix file servers

It supports features such as:

* snapshots
* clones
* low-latency file access

### Example

> "Migrate an on-premises ZFS file server to AWS."

→ **FSx for OpenZFS**

### Remember

> **ZFS → OpenZFS**

---

## FSx for Windows vs EFS

This is an important exam comparison.

|                  | FSx for Windows     | EFS                       |
| ---------------- | ------------------- | ------------------------- |
| Main OS          | **Windows**         | Linux                     |
| Protocol         | **SMB**             | **NFS**                   |
| Active Directory | **Yes**             | No Windows AD integration |
| Main use         | Windows file shares | Linux shared file system  |

### Exam rule

```text
Windows + SMB
→ FSx for Windows

Linux + NFS
→ EFS
```

---

## FSx for Lustre vs EFS

Both can provide shared file storage, but their purposes are different.

|                   | FSx for Lustre                 | EFS                                       |
| ----------------- | ------------------------------ | ----------------------------------------- |
| Main purpose      | **High-performance workloads** | General Linux shared storage              |
| Typical workloads | HPC, ML, rendering             | Web/app servers                           |
| Performance       | Very high                      | General-purpose                           |
| S3 integration    | **Yes**                        | No direct filesystem-style S3 integration |
| Protocol          | Lustre                         | NFS                                       |

### Exam rule

> **Normal Linux shared storage → EFS**

> **HPC / ML / very high-performance → FSx for Lustre**

---

## FSx for NetApp ONTAP vs OpenZFS

|                 | ONTAP                             | OpenZFS              |
| --------------- | --------------------------------- | -------------------- |
| Main keyword    | **NetApp**                        | **ZFS**              |
| Protocols       | NFS, SMB, iSCSI                   | NFS                  |
| Main use        | NetApp migration / multi-protocol | ZFS workloads        |
| Special feature | SnapMirror                        | ZFS snapshots/clones |

### Exam rule

> **NetApp → ONTAP**

> **ZFS → OpenZFS**

---

## The decision algorithm

When you see an FSx question, first look for the **technology or workload**.

```text
Windows / SMB / AD / .NET
→ FSx for Windows

HPC / ML / rendering / very high throughput
→ FSx for Lustre

S3 data needs high-performance file processing
→ FSx for Lustre

NetApp / SnapMirror
→ FSx for NetApp ONTAP

NFS + SMB + iSCSI
→ FSx for NetApp ONTAP

ZFS
→ FSx for OpenZFS

Normal Linux shared file storage
→ EFS
```

---

## Question patterns

> *"Windows applications need shared storage with Active Directory authentication"* → **FSx for Windows File Server**

> *"Application uses SMB and must integrate with Active Directory"* → **FSx for Windows File Server**

> *"ML training requires very high-throughput access to data stored in S3"* → **FSx for Lustre**

> *"HPC workload needs a high-performance parallel file system"* → **FSx for Lustre**

> *"Short-term, temporary high-performance storage for a processing job"* → **FSx for Lustre Scratch**

> *"Long-running Lustre workload needs more data protection"* → **FSx for Lustre Persistent**

> *"Migrate on-premises NetApp storage to AWS"* → **FSx for NetApp ONTAP**

> *"Existing NetApp environment uses SnapMirror"* → **FSx for NetApp ONTAP**

> *"Linux and Windows clients need access to the same file system using NFS and SMB"* → **FSx for NetApp ONTAP**

> *"Migrate an on-premises ZFS file server to AWS"* → **FSx for OpenZFS**

> *"Linux EC2 instances just need a general shared file system"* → **EFS**

---

## Pocket card

| Keyword                                              | Answer                                         |
| ---------------------------------------------------- | ---------------------------------------------- |
| Windows / SMB / NTFS / AD / .NET                     | **FSx for Windows File Server**                |
| HPC / ML / rendering / very high throughput          | **FSx for Lustre**                             |
| Process S3 data using a high-performance file system | **FSx for Lustre**                             |
| Temporary Lustre workload                            | **Lustre Scratch**                             |
| Longer-term Lustre workload                          | **Lustre Persistent**                          |
| NetApp / SnapMirror                                  | **FSx for NetApp ONTAP**                       |
| NFS + SMB + iSCSI                                    | **FSx for NetApp ONTAP**                       |
| ZFS                                                  | **FSx for OpenZFS**                            |
| Normal Linux shared storage                          | **EFS**                                        |
| Linux + NFS                                          | **EFS / OpenZFS depending on the requirement** |
| Windows + SMB                                        | **FSx for Windows**                            |

### Final memory

```text
Windows
→ FSx for Windows

HPC / ML / S3 + high performance
→ FSx for Lustre

NetApp
→ FSx for NetApp ONTAP

ZFS
→ FSx for OpenZFS

Normal Linux shared storage
→ EFS
```

> **The easiest FSx question strategy: find the named technology first.**

Windows? → **Windows FSx**
Lustre/HPC? → **Lustre**
NetApp? → **ONTAP**
ZFS? → **OpenZFS**
Nothing special, just Linux shared storage? → **EFS**
