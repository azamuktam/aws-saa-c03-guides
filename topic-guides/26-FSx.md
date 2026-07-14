# Section 26: FSx Family

## The idea

You already know EFS: the generic, elastic, shared filesystem for Linux. But some workloads don't want "generic" — they were born and raised on a *specific* filesystem technology, and they refuse to run on anything else. A .NET app expects Windows file shares. A supercomputing cluster expects Lustre. A company that spent ten years on NetApp hardware expects NetApp.

FSx is AWS's answer: **fully managed versions of famous third-party filesystems**. Think of it like a food court. EFS is the everyday cafeteria — fine for everyone eating plain Linux food. FSx is the row of specialty restaurants next to it: one only serves Windows, one only serves high-performance computing, one only serves NetApp loyalists, one only serves ZFS fans. You don't pick a restaurant by taste — you pick it because **the exam question names the technology**, and you match the name.

**Key vocab spelled out:** SMB = Server Message Block (the Windows file-sharing protocol). NFS = Network File System (the Linux one). HPC = High-Performance Computing. NTFS = the Windows filesystem format.

## The four specialists

| FSx flavor | Trigger words | What it is |
|---|---|---|
| **FSx for Windows File Server** | Windows, SMB, NTFS, Active Directory, .NET, "user home directories" | Managed Windows file share |
| **FSx for Lustre** | HPC, ML training, video rendering, financial simulation, "process S3 data fast" | Parallel filesystem, hundreds of GB/s throughput |
| **FSx for NetApp ONTAP** | NetApp, SnapMirror, multi-protocol | Managed NetApp; speaks NFS + SMB + iSCSI at once |
| **FSx for OpenZFS** | ZFS, "Linux NFS file server migration" | Managed ZFS with snapshots/clones |

### FSx for Windows File Server
- Native **SMB protocol and NTFS**, integrates with **Active Directory** (both AWS Managed AD and your on-prem AD).
- Supports **Multi-AZ** deployment for high availability.
- THE trap: **EFS is Linux-only — it cannot serve Windows/SMB clients.** The moment you see "Windows," EFS is eliminated.

### FSx for Lustre
- "Lustre" = Linux + cluster. Built for **massively parallel access**: machine learning training, video rendering farms, genomics, financial modeling. Scales to **hundreds of GB/s** and millions of IOPS.
- **Unique superpower: native S3 integration.** Point Lustre at an S3 bucket and it presents the objects as files; results can be written back to S3. "Process data *in S3* with a high-performance filesystem" → Lustre, every time.
- Two deployment types:

| Type | Use | Durability |
|---|---|---|
| **Scratch** | Short-term, temporary processing | No replication — cheap, data lost if hardware fails |
| **Persistent** | Long-term storage | Replicated within AZ |

### FSx for NetApp ONTAP
- The migration landing pad for on-prem **NetApp** appliances. Supports **SnapMirror** replication, so you can mirror on-prem NetApp straight into AWS.
- **Multi-protocol**: NFS, SMB, *and* iSCSI from one filesystem — the only FSx that serves Linux and Windows clients simultaneously.

### FSx for OpenZFS
- For migrating **ZFS-based** or generic NFS Linux file servers, with ZFS goodies: instant **snapshots and clones**, low-latency performance.

**The decision algorithm:** scan the question for a named technology. Windows/SMB/AD → FSx for Windows. HPC or S3-as-a-filesystem → Lustre. NetApp → ONTAP. ZFS → OpenZFS. No named tech, just "shared storage for Linux EC2 instances" → plain EFS.

## Question patterns

> *"Windows applications need shared storage with Active Directory authentication"* → **FSx for Windows File Server** (SMB + AD = Windows FSx; EFS can't do Windows)

> *"ML training job needs high-throughput access to a dataset stored in S3"* → **FSx for Lustre** (only Lustre mounts S3 as a filesystem at HPC speed)

> *"Migrate on-premises NetApp storage to AWS with minimal changes"* → **FSx for NetApp ONTAP** (the word "NetApp" is the whole answer)

> *"Short-term, cost-optimized high-performance storage for a batch processing job"* → **FSx for Lustre Scratch** (temporary + cheap + no replication = Scratch)

> *"File share accessed by both Linux (NFS) and Windows (SMB) clients"* → **FSx for NetApp ONTAP** (the only multi-protocol option)

> *".NET application storing user home directories, must survive AZ failure"* → **FSx for Windows, Multi-AZ** (Windows workload + HA = Multi-AZ Windows FSx)

> *"Replace an on-prem ZFS file server serving NFS to Linux clients"* → **FSx for OpenZFS** (ZFS named → OpenZFS)

## Pocket card

| Keyword | Answer |
|---|---|
| Windows / SMB / NTFS / AD / .NET | FSx for Windows File Server |
| HPC / ML training / rendering / GB/s | FSx for Lustre |
| Process S3 data as files | FSx for Lustre (S3 integration) |
| Temporary + cheapest Lustre | Scratch deployment |
| Long-term Lustre | Persistent deployment |
| NetApp / SnapMirror | FSx for NetApp ONTAP |
| NFS + SMB + iSCSI together | FSx for NetApp ONTAP |
| ZFS | FSx for OpenZFS |
| Plain Linux shared storage | EFS (not FSx at all) |

FSx moves files at specialty speed — but when the data is too big to move over a wire at all, you put it on a truck, which is exactly where the Snow Family comes in.
