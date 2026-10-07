# Section 26: FSx Family

## The idea

**Amazon FSx** provides managed file systems for specific file-system technologies and workloads. AWS offers four main FSx types:

* **FSx for Windows File Server** → Windows workloads
* **FSx for Lustre** → HPC, ML, high-performance/S3 workloads
* **FSx for NetApp ONTAP** → NetApp and multi-protocol storage
* **FSx for OpenZFS** → ZFS/Linux/NFS workloads

```text
Windows / SMB / AD
→ FSx for Windows

HPC / ML / S3 high-performance
→ FSx for Lustre, Lustre protocol
→ primarily Linux-based workloads, not Windows file sharing

NetApp / SnapMirror / NFS + SMB + iSCSI
→ FSx for NetApp ONTAP

ZFS
→ FSx for OpenZFS
```

### Key vocabulary

* **SMB** = Windows file-sharing protocol
* **NFS** = common Linux/Unix file-sharing protocol
* **Lustre** = high-performance parallel file-system protocol
* **iSCSI** = block-storage protocol over IP
* **HPC** = High-Performance Computing
* **NTFS** = Windows file system
* **SnapMirror** = NetApp replication technology
* **Snapshot** = point-in-time copy of a file system/data state

---

# The four specialists

| FSx type                        | Main keywords                                    | Main use                        |
| ------------------------------- | ------------------------------------------------ | ------------------------------- |
| **FSx for Windows File Server** | Windows, SMB, NTFS, AD, .NET                     | Windows file shares             |
| **FSx for Lustre**              | HPC, ML, rendering, high throughput, S3          | High-performance workloads      |
| **FSx for NetApp ONTAP**        | NetApp, SnapMirror, snapshots, NFS, SMB, iSCSI   | NetApp / multi-protocol storage |
| **FSx for OpenZFS**             | ZFS, NFS, snapshots, Linux file-server migration | ZFS/Linux workloads             |

---

# FSx for Windows File Server

Managed Windows file system using:

* **SMB**
* **NTFS**
* **Active Directory**

```text
Windows application
       ↓ SMB
FSx for Windows
```

Can integrate with:

* AWS Managed Microsoft AD
* On-premises AD

Supports **Multi-AZ** deployment for HA.

### Use when

* Windows applications need shared storage
* SMB + Active Directory
* Windows file shares

> **Windows + SMB + AD → FSx for Windows**

### EFS distinction

```text
Windows + SMB
→ FSx for Windows

Linux + NFS
→ EFS
```

---

# FSx for Lustre

Designed for **very high-performance workloads**:

* HPC
* ML training
* Video rendering
* Financial simulations
* Genomics
* Very high throughput / IOPS workloads

### Protocol

**FSx for Lustre uses the Lustre protocol.**

It is intended for **Linux-based high-performance workloads**, not Windows SMB file sharing.

```text
Linux / HPC client
       ↓ Lustre
FSx for Lustre
```

> **Lustre protocol → FSx for Lustre**

> **Windows + SMB → FSx for Windows**

### S3 integration

Can process data stored in S3 through a high-performance Lustre file system.

```text
S3
 ↓
FSx for Lustre
 ↓
HPC / ML
```

> **HPC / ML + high performance + S3 → FSx for Lustre**

## Lustre deployment types

| Type           | Use                             | Data protection           |
| -------------- | ------------------------------- | ------------------------- |
| **Scratch**    | Temporary/short-term processing | No replication            |
| **Persistent** | Longer-term workloads           | Data replicated within AZ |

### Scratch

Use for:

* Temporary data
* Short-term jobs
* Maximum performance
* Lower cost

Data can be lost if the scratch file system fails.

### Persistent

Use for:

* Longer-running workloads
* More data protection
* Non-temporary processing

> **Temporary → Scratch**

> **Longer-term → Persistent**

---

# FSx for Lustre vs EFS

|                | **FSx for Lustre**         | **EFS**                                   |
| -------------- | -------------------------- | ----------------------------------------- |
| Main purpose   | High-performance workloads | General Linux shared storage              |
| Workloads      | HPC, ML, rendering         | Web/app servers                           |
| Performance    | Very high                  | General-purpose                           |
| S3 integration | **Yes**                    | No direct filesystem-style S3 integration |
| Protocol       | **Lustre**                 | **NFS**                                   |
| Windows SMB    | **No**                     | No                                        |

> **Normal Linux shared storage → EFS**

> **HPC / ML / very high performance → FSx for Lustre**

---

# FSx for NetApp ONTAP

Managed **NetApp** file system.

Useful when migrating existing NetApp environments to AWS.

Supports:

* **NFS**
* **SMB**
* **iSCSI**

This makes it **multi-protocol**. ONTAP can also use Active Directory for SMB access.

### SnapMirror

NetApp **SnapMirror** supports replication between NetApp systems.

Useful for:

* Migration
* Replication
* Disaster recovery

```text
NetApp system
      ↓
   SnapMirror
      ↓
FSx for NetApp ONTAP
```

> **NetApp + SnapMirror → FSx for NetApp ONTAP**

### Snapshots

ONTAP also supports **point-in-time snapshots**.

Useful for:

* Fast recovery
* Recovering files/data
* Restoring earlier file versions

> **ONTAP + SnapMirror → replication**

> **ONTAP + snapshots → point-in-time recovery**

### Multi-protocol example

```text
Linux → NFS
Windows → SMB
iSCSI clients → iSCSI
       ↓
FSx for NetApp ONTAP
```

> **NFS + SMB + iSCSI → ONTAP**

---

# FSx for OpenZFS

Managed **OpenZFS** file system.

Useful for:

* ZFS workloads
* NFS
* Linux/Unix file-server migration

Supports:

* Snapshots
* Clones
* Low-latency file access

OpenZFS supports NFS versions 3, 4.0, 4.1, and 4.2.

### Snapshots

FSx for OpenZFS supports **point-in-time snapshots** of volumes.

Snapshots can also be used to create clone volumes.

> **ZFS + snapshots/clones → FSx for OpenZFS**

> On-premises ZFS → **FSx for OpenZFS**

---

# FSx for NetApp ONTAP vs OpenZFS

|                 | **ONTAP**               | **OpenZFS**          |
| --------------- | ----------------------- | -------------------- |
| Main keyword    | **NetApp**              | **ZFS**              |
| Protocols       | NFS, SMB, iSCSI         | NFS                  |
| Main use        | NetApp / multi-protocol | ZFS workloads        |
| Special feature | SnapMirror + snapshots  | ZFS snapshots/clones |

> **NetApp → ONTAP**

> **SnapMirror → ONTAP**

> **ZFS → OpenZFS**

> **Snapshot alone → can be either ONTAP or OpenZFS**

---

# FSx for Windows vs EFS

|                  | **FSx for Windows** | **EFS**                   |
| ---------------- | ------------------- | ------------------------- |
| Main OS          | **Windows**         | Linux                     |
| Protocol         | **SMB**             | **NFS**                   |
| Active Directory | **Yes**             | No Windows AD integration |
| Main use         | Windows file shares | Linux shared storage      |

---

# The decision algorithm

```text
Windows / SMB / AD / .NET
→ FSx for Windows

HPC / ML / rendering / very high throughput
→ FSx for Lustre

S3 data + high-performance processing
→ FSx for Lustre

Lustre protocol
→ FSx for Lustre

NetApp / SnapMirror
→ FSx for NetApp ONTAP

NFS + SMB + iSCSI
→ FSx for NetApp ONTAP

ONTAP snapshots / SnapMirror
→ FSx for NetApp ONTAP

ZFS / NFS / snapshots + clones
→ FSx for OpenZFS

Normal Linux shared storage
→ EFS
```

---

# Question patterns

> **"Windows applications need shared storage with Active Directory authentication."**

→ **FSx for Windows File Server**

> **"Application uses SMB and must integrate with Active Directory."**

→ **FSx for Windows File Server**

> **"ML training needs very high-throughput access to data stored in S3."**

→ **FSx for Lustre**

> **"HPC workload needs a high-performance parallel file system."**

→ **FSx for Lustre**

> **"The workload uses the Lustre protocol."**

→ **FSx for Lustre**

> **"Windows-based applications require SMB file sharing."**

→ **FSx for Windows File Server**

**Not → FSx for Lustre**

> **"Short-term temporary high-performance processing."**

→ **FSx for Lustre Scratch**

> **"Long-running Lustre workload needs more data protection."**

→ **FSx for Lustre Persistent**

> **"Migrate on-premises NetApp storage to AWS."**

→ **FSx for NetApp ONTAP**

> **"Existing NetApp environment uses SnapMirror."**

→ **FSx for NetApp ONTAP**

> **"The storage solution needs NetApp SnapMirror replication."**

→ **FSx for NetApp ONTAP**

> **"The file system needs point-in-time snapshots."**

→ **ONTAP or OpenZFS, depending on the other requirements**

> **"Linux and Windows clients need the same file system using NFS and SMB."**

→ **FSx for NetApp ONTAP**

> **"Migrate an on-premises ZFS file server to AWS."**

→ **FSx for OpenZFS**

> **"Linux EC2 instances need a general shared file system."**

→ **EFS**

---

# Pocket card

| Keyword                                     | Answer                                     |
| ------------------------------------------- | ------------------------------------------ |
| Windows / SMB / NTFS / AD / .NET            | **FSx for Windows File Server**            |
| HPC / ML / rendering / very high throughput | **FSx for Lustre**                         |
| Lustre protocol                             | **FSx for Lustre**                         |
| Windows + SMB                               | **FSx for Windows**                        |
| S3 + high-performance processing            | **FSx for Lustre**                         |
| Temporary Lustre workload                   | **Lustre Scratch**                         |
| Longer-term Lustre workload                 | **Lustre Persistent**                      |
| NetApp / SnapMirror                         | **FSx for NetApp ONTAP**                   |
| NetApp snapshots                            | **FSx for NetApp ONTAP**                   |
| NFS + SMB + iSCSI                           | **FSx for NetApp ONTAP**                   |
| ZFS                                         | **FSx for OpenZFS**                        |
| OpenZFS snapshots / clones                  | **FSx for OpenZFS**                        |
| Snapshot alone                              | **ONTAP or OpenZFS**                       |
| Normal Linux shared storage                 | **EFS**                                    |
| Linux + NFS                                 | **EFS / OpenZFS depending on requirement** |
| Windows + SMB                               | **FSx for Windows**                        |
