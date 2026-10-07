# Section 27: Snow Family

## The idea

**AWS Snow Family = physical AWS devices for moving or processing large amounts of data when network transfer is too slow or impractical.**

Example:

```text
Company
   ↓
Snow device
   ↓
Ship to AWS
   ↓
S3
```

Instead of sending huge amounts of data over the internet, **physically move the device**.

---

## The device lineup

| Device                              | Capacity                              | When                                 |
| ----------------------------------- | ------------------------------------- | ------------------------------------ |
| **Snowcone**                        | 8–14 TB                               | Small, rugged, remote/edge sites     |
| **Snowball Edge Storage Optimized** | ~80 TB usable                         | Large data migrations                |
| **Snowball Edge Compute Optimized** | Less storage, more CPU + optional GPU | Edge computing                       |
| **Snowmobile**                      | Up to **100 PB**                      | Huge transfers, typically **>10 PB** |

### AWS Data Transfer Terminal

**Data Transfer Terminal = AWS facility where you bring your own storage devices and transfer data to AWS over a high-speed connection.**

```text
Your storage devices
        ↓
AWS Data Transfer Terminal
        ↓
AWS
```

Key difference:

* **Snowball Edge** → AWS gives you the device.
* **Data Transfer Terminal** → **you bring your own storage devices.**
* **DataSync** → network-based transfer.

---

## Extra points

* **Snowcone** has a preinstalled **DataSync agent**.
* Multiple Snowballs can be used for large migrations.
* **AWS OpsHub** = GUI for managing Snow devices.

---

## The process

1. Order the device.
2. AWS ships it to you.
3. Copy data to it; data is encrypted.
4. Ship it back.
5. AWS imports the data into S3.

### THE trap

**Snowball cannot import directly into S3 Glacier.**

Data goes to **S3 first**, then an **S3 lifecycle rule** can move it to Glacier.

---

## Edge computing angle

**Snowball Edge Compute Optimized** can run **EC2 and Lambda locally**.

Think:

> Remote location + little/no connectivity + need to process data locally

---

## Question patterns

> *"Migrate 200 TB, the site has a 100 Mbps link, deadline in 3 weeks"* → **Snowball Edge Storage Optimized** (network math = months, so ship devices — a few 80 TB units)

> *"Decommission an entire datacenter: multiple petabytes (>10 PB) to AWS"* → **Snowmobile** (past ~10 PB, send the truck)

> *"Research ship must run analysis on collected data with no internet connectivity"* → **Snowball Edge Compute Optimized** (compute at the disconnected edge; GPU if ML is mentioned)

> *"Company wants to archive 80 TB directly into S3 Glacier using Snowball"* → **Import to S3, then lifecycle rule to Glacier** (Snowball can't write to Glacier directly — THE trap)

> *"Small remote clinic needs to transfer ~8–10 TB from a space- and power-constrained site"* → **Snowcone** (tiny footprint, tiny capacity)

> *"Transfer 40 TB once; would take 6 weeks over the existing connection"* → **Snowball Edge** (>1 week over the wire → Snow family)

> *"Manage Snow devices with a graphical interface"* → **AWS OpsHub** (the Snow GUI)

> *"Company already has its own storage devices and wants to physically bring them to AWS for a high-speed bulk transfer"* → **AWS Data Transfer Terminal**

> *"On-premises file servers need to migrate/synchronize data over the network"* → **AWS DataSync**, not Data Transfer Terminal

---

## Pocket card

| Keyword                                                                | Answer                          |
| ---------------------------------------------------------------------- | ------------------------------- |
| Transfer would take > 1 week                                           | Snow family                     |
| 8–14 TB, tiny/rugged/edge                                              | Snowcone                        |
| 50–500 TB migration                                                    | Snowball Edge Storage Optimized |
| Process data offline / GPU at edge                                     | Snowball Edge Compute Optimized |
| > 10 PB, up to 100 PB                                                  | Snowmobile                      |
| Already have your own storage devices and physically bring them to AWS | **Data Transfer Terminal**      |
| AWS sends you a device to load and return                              | **Snowball Edge**               |
| Transfer over the network / migration / synchronization                | **DataSync**                    |
| Straight to Glacier?                                                   | No — S3 first + lifecycle rule  |
| Encryption on device                                                   | KMS, automatic                  |
| GUI for Snow devices                                                   | OpsHub                          |
| Preinstalled DataSync agent                                            | Snowcone                        |

Once your data (and everything else) is in AWS, you'll want to build environments the same way twice without clicking — that's CloudFormation's whole reason to exist.
