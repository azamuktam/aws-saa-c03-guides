# Section 27: Snow Family

## The idea

**AWS Snow Family = physical AWS devices that help you move or process huge amounts of data when using the internet is too slow or impractical.**

## Why do we need it?

Imagine your company has **500 TB of data** on-premises.

You could upload it:

```text
Company → Internet → S3

But with a slow connection, this could take a very long time.

Instead, AWS can send you a Snow device:

Company
   ↓
Snow device
   ↓
Ship it to AWS
   ↓
S3
```
So the basic idea is:

Instead of sending the data through the internet, physically send the storage device to AWS.


## The device lineup

| Device | Capacity | When |
|---|---|---|
| **Snowcone** | 8–14 TB | Tiny, rugged, ships anywhere (even drone-deliverable); small edge sites |
| **Snowball Edge Storage Optimized** | ~80 TB usable | **The workhorse** — migrations of ~50–500 TB (order several) |
| **Snowball Edge Compute Optimized** | Less storage, more CPU + **optional GPU** | Edge processing: run EC2/Lambda where there's no connectivity |
| **Snowmobile** | Up to **100 PB** | An actual shipping-container truck; consider at **>10 PB** |

Extra flavor points:
- **Snowcone** comes with the **DataSync agent preinstalled** — copy data on it, and it can sync back online later, or you ship it.
- Multiple Snowballs can be ordered in parallel for big jobs (e.g., 200 TB ≈ 3 × 80 TB Snowball Edge Storage Optimized).
- **AWS OpsHub** = the **GUI application** for managing Snow devices (no CLI wrestling required).

## The process

1. **Order** the device in the AWS console.
2. AWS **ships** it to you.
3. You connect it locally and **copy data — encrypted automatically with KMS** (Key Management Service). Keys never live on the device in usable form.
4. **Ship it back** (E Ink label updates itself).
5. AWS **loads the data into S3**, then securely wipes the device.

THE trap: **Snowball cannot import directly into S3 Glacier.** Data always lands in **S3 first**; if you want Glacier, you attach an **S3 lifecycle rule** that transitions the objects afterward. Any answer choice saying "import from Snowball straight to Glacier" is wrong — reliably tested.

**Edge computing angle:** Snowball Edge Compute Optimized (with optional GPU) runs EC2 AMIs and Lambda locally. Scenario smells: "vessel collecting sensor data with intermittent connectivity," "remote facility must process video before transfer." Process on the device, ship or sync results later.

## Question patterns

> *"Migrate 200 TB, the site has a 100 Mbps link, deadline in 3 weeks"* → **Snowball Edge Storage Optimized** (network math = months, so ship devices — a few 80 TB units)

> *"Decommission an entire datacenter: multiple petabytes (>10 PB) to AWS"* → **Snowmobile** (past ~10 PB, send the truck)

> *"Research ship must run analysis on collected data with no internet connectivity"* → **Snowball Edge Compute Optimized** (compute at the disconnected edge; GPU if ML is mentioned)

> *"Company wants to archive 80 TB directly into S3 Glacier using Snowball"* → **Import to S3, then lifecycle rule to Glacier** (Snowball can't write to Glacier directly — THE trap)

> *"Small remote clinic needs to transfer ~8–10 TB from a space- and power-constrained site"* → **Snowcone** (tiny footprint, tiny capacity)

> *"Transfer 40 TB once; would take 6 weeks over the existing connection"* → **Snowball Edge** (>1 week over the wire → Snow family)

> *"Manage Snow devices with a graphical interface"* → **AWS OpsHub** (the Snow GUI)

## Pocket card

| Keyword | Answer |
|---|---|
| Transfer would take > 1 week | Snow family |
| 8–14 TB, tiny/rugged/edge | Snowcone |
| 50–500 TB migration | Snowball Edge Storage Optimized |
| Process data offline / GPU at edge | Snowball Edge Compute Optimized |
| > 10 PB, up to 100 PB | Snowmobile |
| Straight to Glacier? | No — S3 first + lifecycle rule |
| Encryption on device | KMS, automatic |
| GUI for Snow devices | OpsHub |
| Preinstalled DataSync agent | Snowcone |

Once your data (and everything else) is in AWS, you'll want to build environments the same way twice without clicking — that's CloudFormation's whole reason to exist.
