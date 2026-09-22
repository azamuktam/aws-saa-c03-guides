# Section 6: VPC — Virtual Private Cloud

## The idea

A **VPC (Virtual Private Cloud)** is your private network inside AWS.

You control:

* IP address ranges
* Resource placement
* Internet access
* Communication between resources/networks
* Routing

```text id="s3k3h2"
AWS Region
└── VPC
    ├── Subnet → EC2
    └── Subnet → Database
```

> **VPC = your AWS network**

---

# Boxes in boxes

```text id="0v4f6e"
AWS Region
└── VPC
    ├── AZ-A
    │   ├── Public Subnet → EC2
    │   └── Private Subnet → EC2
    │
    └── AZ-B
        ├── Public Subnet → EC2
        └── Private Subnet → Database
```

* VPC → one Region
* Subnet → one AZ
* Multi-AZ → multiple subnets
* Subnets contain resources
* **Route tables determine traffic paths**
* newly created subnet is, by default, linked to the **main route table** of the VPC.

> Route table = **"Traffic to X goes through Y."**

---

# Route tables

Example:

```text id="s0v5k6"
Destination       Target
0.0.0.0/0         Internet Gateway
10.0.0.0/16       local
```

The `local` route allows communication within the VPC CIDR, subject to security controls.

## Longest-prefix match

When multiple routes match, the **most specific route wins**.

```text id="g3p4jq"
10.0.0.0/16 → Transit Gateway
10.0.1.0/24 → VPC Peering
```

Traffic to `10.0.1.50` uses `/24`.

> **More specific route wins.**

---

# Public vs private

A subnet is **public** when its route table has a route to an **Internet Gateway (IGW)**.

```text id="22wh0h"
0.0.0.0/0 → Internet Gateway
```

A subnet is private when it has **no route to an IGW**.

> **Public subnet = route to IGW**

Not the subnet name and not simply having a public IP.

## Internet Gateway

Provides internet connectivity:

```text id="9l8w49"
EC2 ↔ IGW ↔ Internet
```

For IPv4, both directions are possible if routing, public IP addressing, and security rules allow it.

---

# Public subnet vs internet-reachable EC2

For IPv4 Internet reachability, an EC2 instance normally needs:

```text id="7v5d2t"
Public subnet
+
Public IPv4 / Elastic IP
+
Allowed SG/NACL traffic
```

Therefore:

```text id="3np5yk"
Public subnet
= route to IGW

Internet-reachable EC2
= public subnet + public IP + allowed traffic
```

---

# Internet access components

| Component            | IP          | Traffic  | Purpose                              |
| -------------------- | ----------- | -------- | ------------------------------------ |
| **Internet Gateway** | IPv4 + IPv6 | Two-way  | Internet access for public resources |
| **NAT Gateway**      | IPv4        | Outbound | Private IPv4 → Internet              |
| **Egress-Only IGW**  | IPv6        | Outbound | Private IPv6 → Internet              |
| **VPC Endpoint**     | IPv4/IPv6   | Private  | Access AWS services privately        |

---

# NAT Gateway — private IPv4 outbound

Used when private instances need:

* OS/package updates
* External APIs
* Internet access

```text id="te1mb2"
Private EC2
 ↓
NAT Gateway
 ↓
Internet Gateway
 ↓
Internet
```

```text id="l1v2g1"
Private EC2 → Internet ✅
Internet → Private EC2 ❌
```

## NAT Gateway placement

A traditional zonal NAT Gateway is in a **public subnet**.

Private subnet route:

```text
0.0.0.0/0 → NAT Gateway
```

NAT then uses the IGW.

## NAT availability

Traditional high-availability pattern:

```text id="m99eqx"
AZ-A → NAT Gateway A
AZ-B → NAT Gateway B
```

Avoid making one AZ's NAT Gateway a dependency for another AZ.

### Current AWS nuance

AWS also provides **Regional NAT Gateways**, which automatically expand across AZs.

For SAA:

> **Traditional HA design → one zonal NAT Gateway per AZ**

> **Regional NAT Gateway → managed cross-AZ expansion**

---

# NAT Gateway vs NAT Instance

| NAT Gateway                   | NAT Instance                      |
| ----------------------------- | --------------------------------- |
| AWS-managed                   | EC2 you manage                    |
| Automatically scales          | You manage scaling/patching       |
| No security group             | Can use security group            |
| Hourly + data processing cost | EC2-based cost                    |
| —                             | Disable source/destination checks |

---

# IPv6

NAT Gateway is mainly an **IPv4** solution.

For IPv6 outbound-only access:

> **Egress-Only Internet Gateway**

```text id="9ck3u3"
IPv6 EC2
 ↓
Egress-Only IGW
 ↓
Internet
```

```text id="9n7hi1"
EC2 → Internet ✅
Internet → EC2 ❌
```

> **NAT Gateway → IPv4 outbound**

> **Egress-Only IGW → IPv6 outbound**

---

# CIDR essentials

CIDR describes an IP range.

```text
10.0.0.0/16
```

IPv4 = **32 bits = 4 × 8-bit octets**.

`/16` means the first 16 bits are fixed:

```text
10.0.X.X
```

Range:

```text
10.0.0.0 → 10.0.255.255
```

Common sizes:

```text
/16 → 65,536 addresses
/24 → 256 addresses
```

### Important facts

* VPC CIDR: **/16 to /28**
* AWS reserves **5 IPv4 addresses per subnet**
* `/24` subnet → `256 - 5 = 251 usable`

---

# Lambda + VPC capacity

A VPC-connected Lambda function that scales heavily can run into **ENI or subnet IP capacity** limits.

Possible symptoms include:

* `EC2ThrottledException`
* `SubnetIPAddressLimitReachedException`
* ENI/IP exhaustion during VPC initialization

> **Lambda + VPC + high concurrency → think ENIs + subnet IP capacity**

Example:

```text id="k7f6qn"
10.31.0.0/27
→ 32 total IPv4 addresses
→ 27 usable AWS subnet IPs
```

A small subnet can therefore become a scaling bottleneck for a high-concurrency Lambda workload.

---

# No overlapping CIDRs

Networks that need to connect should use **non-overlapping CIDRs**.

Example:

```text
VPC-A: 10.0.0.0/16
VPC-B: 10.0.0.0/16
```

Overlapping ranges cause problems with:

* VPC Peering
* Transit Gateway routing
* VPN connectivity

> **Non-overlapping CIDRs make connectivity easier.**

---

# Security Groups vs NACLs

|              | **Security Group**   | **NACL**            |
| ------------ | -------------------- | ------------------- |
| Applies to   | **ENI / instance**   | **Subnet**          |
| Stateful     | **Yes**              | **No**              |
| Rules        | **Allow only**       | Allow + Deny        |
| Evaluation   | All applicable rules | Lowest number first |
| SG reference | **Yes**              | No, CIDR-based      |

---

# Security Group

Security Groups are **stateful**.

```text id="3pdxw0"
EC2 → Database
```

If outbound traffic is allowed, the response is automatically allowed.

### SG-to-SG example

```text id="e14gxt"
ALB SG
 ↓
App SG
 ↓
DB SG
```

For MySQL:

```text
DB SG:
TCP 3306
Source: App SG
```

This is better than fixed IPs when application instance IPs change.

### Important

Security Groups **cannot DENY**.

> **Block a specific IP → NACL**

---

# NACL

NACLs are **stateless**.

Both directions must be allowed:

```text id="5fjxzh"
Client → Server
Server → Client
```

Rules are evaluated from the **lowest number upward**; first match wins.

Example:

```text id="5jly6a"
100 → DENY 10.0.0.5/32
200 → ALLOW 0.0.0.0/0
```

`10.0.0.5` is denied by rule 100.

---

# Ephemeral ports

Responses commonly use ephemeral ports:

```text
1024–65535
```

NACLs may need these ports for return traffic.

> **Request leaves but response does not return → check NACL / ephemeral ports**

---

# Security Group vs NACL memory

```text id="2v4d9h"
Security Group
= stateful
= instance / ENI
= allow only
= SG references

NACL
= stateless
= subnet
= allow + deny
= CIDR
= lowest rule number first
```

---

# VPC Endpoints

VPC Endpoints provide a **private path to AWS services without the public Internet**.

|                          | **Gateway Endpoint** | **Interface Endpoint**   |
| ------------------------ | -------------------- | ------------------------ |
| Main services            | **S3, DynamoDB**     | Many AWS services + SaaS |
| Cost                     | **Free**             | Paid                     |
| Implementation           | Route table          | ENI                      |
| On-prem / peering access | No                   | Yes                      |

---

# Gateway Endpoint

Used for:

* S3
* DynamoDB

```text id="4x7p5k"
Private EC2
 ↓
Gateway Endpoint
 ↓
S3
```

No NAT required.

> **Private S3/DynamoDB + lowest cost → Gateway Endpoint**

Gateway endpoints use route tables.

---

# Interface Endpoint

Uses an **ENI** in your subnet and **AWS PrivateLink**.

Supports many AWS services and SaaS and can be used through:

* VPC Peering
* Transit Gateway
* VPN
* On-premises connectivity

### Important nuance

S3 and DynamoDB also support Interface Endpoints.

```text id="db1h7v"
S3/DynamoDB
+ normal VPC access + lowest cost
→ Gateway Endpoint

S3/DynamoDB
+ Interface Endpoint capabilities
→ Interface Endpoint
```

> **Gateway → S3/DynamoDB + cheap**

> **Interface → ENI + PrivateLink + many services / advanced connectivity**

---

# Connecting VPCs

```text id="e4d8um"
VPC Peering
= direct VPC-to-VPC

Transit Gateway
= central networking hub

PrivateLink
= access to one service
```

---

# VPC Peering

Connects **two VPCs directly**.

```text id="wllc9e"
VPC-A ←→ VPC-B
```

* Cross-account supported
* Cross-Region supported
* **Not transitive**
* CIDRs cannot overlap

Example:

```text
VPC-A ↔ VPC-B
VPC-B ↔ VPC-C
```

Does not create:

```text
VPC-A ↔ VPC-C
```

> **VPC Peering = direct, not transit**

---

# Transit Gateway

Central hub for:

* VPCs
* Site-to-Site VPN
* Direct Connect

```text id="8s1qv1"
VPC-A
   \
VPC-B — Transit Gateway — VPC-C
   /
 VPN
   |
On-premises
```

* Supports transitive routing
* Can be shared across accounts using **AWS RAM**

> **Many VPCs/networks → Transit Gateway**

---

# PrivateLink

PrivateLink exposes **one service**, not a whole network.

```text id="9q3d1l"
Provider VPC
 ↓
NLB
 ↓
Endpoint Service
 ↓
PrivateLink
 ↓
Customer VPC
```

* Customer accesses the service, not the provider network
* Can work with **overlapping CIDRs**

> **One service/API to another VPC → PrivateLink**

Common use: SaaS service consumed privately by many customer VPCs.

---

# Quick chooser

```text id="e89oij"
2 VPCs → VPC Peering

Many VPCs / VPN / DX → Transit Gateway

One service → PrivateLink
```

---

# Observability & admin access

## VPC Flow Logs

Record **traffic metadata**, including:

* Source IP
* Destination IP
* Port
* Protocol
* ACCEPT / REJECT

They do **not** capture packet contents.

> **Why is traffic rejected? → Flow Logs**

> **Flow Logs = metadata**

---

# Traffic Mirroring

Copies **actual packets** to an inspection/monitoring system.

> **Need packet contents → Traffic Mirroring**

```text id="bq99q3"
Flow Logs
= metadata

Traffic Mirroring
= packets
```

---

# Bastion Host

EC2 jump server for private instances.

```text id="of7avh"
Your computer
 ↓ SSH
Bastion
 ↓ SSH
Private EC2
```

Typical SG:

```text
Bastion SG
→ SSH only from trusted IP

Private EC2 SG
→ SSH from Bastion SG
```

You manage the bastion.

---

# SSM Session Manager

Modern alternative to bastion.

Provides private EC2 access without:

* Public IP
* Port 22
* Bastion

```text id="v1b38y"
You
 ↓
SSM Session Manager
 ↓
Private EC2
```

Sessions can be logged.

> **Most secure private EC2 administration → SSM Session Manager**

---

# VPC Sharing

Allows one account to own the VPC while other accounts use its subnets.

```text id="m4r7va"
Network Account
      ↓
     VPC
   /     \
Account A Account B
```

Uses **AWS RAM**.

> Central networking account + multiple application accounts → **VPC Sharing**

---

# Hybrid Networking & Connectivity

Main services:

* **Direct Connect (DX)** → dedicated private connection
* **Site-to-Site VPN** → encrypted Internet connection
* **VPC Peering** → two VPCs
* **Transit Gateway** → many networks
* **VPN CloudHub** → multiple remote networks via VPN

---

# Direct Connect and VIF

```text id="d0hlyj"
On-premises
    |
Direct Connect
    ↓
AWS
    |
   VIF
```

## Direct Connect connection

Physical/dedicated network connection between on-premises and AWS.

## VIF

Logical connection over Direct Connect using **BGP** for routes.

> **DX = physical connection**

> **VIF = logical connection**

---

# Private VIF

Private VIF connects through a **Virtual Private Gateway (VGW)**.

```text id="n3paj8"
On-premises
 ↓
Direct Connect
 ↓
Private VIF
 ↓
VGW
 ↓
VPC
```

> **Private VIF → VGW / VPC**

---

# Direct Connect Gateway

Centralizes Direct Connect connectivity across supported architectures.

```text id="qv2ao8"
On-premises
 ↓
Direct Connect
 ↓
VIF
 ↓
Direct Connect Gateway
 ├── VPC / VGW
 └── Transit Gateway
```

For Transit Gateway, use a **Transit VIF** with the Direct Connect Gateway.

```text id="mxy5rv"
Private VIF
→ VGW / VPC

Transit VIF
→ DX Gateway
→ Transit Gateway
```

> **One DX architecture + multiple VPCs / centralized TGW → Direct Connect Gateway**

---

# Direct Connect redundancy

One DX connection can be a single point of failure.

Use:

* Another independent DX connection
* Or VPN as backup

```text id="b9f6y4"
DX #1 ──→ VPC
DX #2 ──→ VPC

or

DX ──→ VPC
VPN ─→ VPC
```

> **DX + VPN = independent paths**

---

# Why Private VIF targets the correct VPC

A Private VIF connects to a **VGW**, and the VGW belongs to a specific VPC.

```text id="rfm7f7"
On-premises
 ↓
Private VIF
 ↓
VGW
 ↓
VPC-1
```

The VIF must reach the **correct VGW/VPC**.

> Redundant DX for VPC-1 → ensure the VIF actually connects to **VPC-1**

---

# VPN as backup

```text id="u24k7t"
On-premises
   | \
   |  \ VPN
   DX  \
   |    \
   +----> VPC
```

> **DX failure → VPN can maintain connectivity**

---

# VPC Peering

Private connection between two VPCs:

```text id="6vw13y"
VPC-1 ←→ VPC-2
```

Not transitive.

```text id="xwhw6d"
VPC-1 ←→ VPC-2 ←→ VPC-3
```

does not connect VPC-1 to VPC-3 through VPC-2.

---

# Exam question pattern

Given:

```text id="4w60be"
On-premises
     ↓
Direct Connect
     ↓
VPC-1
```

Need more fault tolerance:

**Good:**

1. Add Site-to-Site VPN directly to VPC-1
2. Add another independent DX connection to VPC-1

**Bad assumption:**

```text id="m6qv8y"
On-premises → VPC-2 → Peering → VPC-1
```

VPC Peering is **not transitive**.

---

# Direct Connect vs VPN

|            | **Direct Connect**              | **Site-to-Site VPN**         |
| ---------- | ------------------------------- | ---------------------------- |
| Connection | Dedicated                       | Internet                     |
| Encryption | Not automatically encrypted     | IPsec encrypted              |
| Main use   | Consistent private connectivity | Secure connectivity / backup |
| Redundancy | Multiple DX connections         | Redundant VPN                |

---

# VPC Peering vs Transit Gateway

|              | **VPC Peering**      | **Transit Gateway** |
| ------------ | -------------------- | ------------------- |
| Connect VPCs | Yes                  | Yes                 |
| Transitive   | **No**               | **Yes**             |
| Model        | Direct               | Central hub         |
| Best for     | Small number of VPCs | Many VPCs/networks  |

---

# VPN CloudHub

Connects multiple remote networks through a hub-and-spoke VPN architecture.

```text id="a6r4ml"
Branch A
   \
    VPN hub
   /
Branch B
```

> **Multiple remote VPN sites → VPN CloudHub**

---

# One-line mental model

```text id="wnp17z"
VPC
= AWS private network

IGW
= Internet connection

NAT Gateway
= Private IPv4 → Internet

Egress-Only IGW
= Private IPv6 → Internet, no inbound

Security Group
= Stateful firewall for instance / ENI

NACL
= Stateless firewall for subnet

Gateway Endpoint
= Private S3/DynamoDB

Interface Endpoint
= Private AWS service/SaaS access

VPC Peering
= Two VPCs

Transit Gateway
= Many networks

PrivateLink
= One service

Flow Logs
= Network metadata

Traffic Mirroring
= Packet copies

SSM Session Manager
= Private EC2 administration
```

# Hybrid Networking — one-line mental model

```text id="1gxz2q"
Direct Connect
= Dedicated connection

Direct Connect Gateway
= DX centralization

Private VIF
= DX → VGW / VPC

Transit VIF
= DX → DXGW → TGW

Site-to-Site VPN
= Encrypted Internet connection

DX + VPN
= Redundancy

VPC Peering
= Direct VPC-to-VPC

Transit Gateway
= Central hub

VPN CloudHub
= Multiple remote VPN networks
```

---

# Question patterns

> **Block a specific IP** → **NACL DENY**

> **Only app servers connect to DB** → **DB SG allows App SG**

> **Private EC2 → private/free S3** → **Gateway Endpoint**

> **25 VPCs + on-premises** → **Transit Gateway**

> **One service to another VPC** → **PrivateLink**

> **Overlapping CIDRs + one service must be shared** → **PrivateLink**

> **Traditional NAT HA across AZs** → **One zonal NAT Gateway per AZ**

> **Private IPv4 EC2 → Internet** → **NAT Gateway**

> **IPv6 outbound-only** → **Egress-Only IGW**

> **Accepted/rejected traffic** → **VPC Flow Logs**

> **Full packet inspection** → **Traffic Mirroring**

> **Secure private EC2 administration** → **SSM Session Manager**

> **NAT Instance not forwarding** → **Disable source/destination check**

> **Requests leave but responses don't return** → **Check NACL ephemeral ports**

> **One DX connection needs fault tolerance** → **Another DX or Site-to-Site VPN**

> **On-prem → specific VPC** → **DX/VPN path must actually reach that VPC**

> **One DX design → multiple VPCs / centralized TGW** → **Direct Connect Gateway**

> **Multiple remote VPN networks** → **VPN CloudHub**

> **Multiple matching routes** → **Longest-prefix match**

---

# Pocket card

| Keyword                         | Answer                                      |
| ------------------------------- | ------------------------------------------- |
| Public subnet                   | Route to **IGW**                            |
| Internet-reachable EC2          | Public subnet + public IP + allowed traffic |
| Private IPv4 → Internet         | **NAT Gateway**                             |
| Traditional NAT HA              | **One NAT Gateway per AZ**                  |
| NAT Instance                    | Disable **source/destination check**        |
| IPv6 outbound-only              | **Egress-Only IGW**                         |
| Reserved IPv4/subnet            | **5**                                       |
| VPC CIDR                        | **/16–/28**                                 |
| Most specific route             | **Longest-prefix match**                    |
| Block traffic                   | **NACL DENY**                               |
| Stateful firewall               | **Security Group**                          |
| Stateless firewall              | **NACL**                                    |
| SG-to-SG access                 | **SG reference**                            |
| S3/DynamoDB private access      | **Gateway Endpoint**                        |
| Other AWS/SaaS                  | **Interface Endpoint**                      |
| Two VPCs                        | **VPC Peering**                             |
| Many VPCs/networks              | **Transit Gateway**                         |
| One service                     | **PrivateLink**                             |
| Traffic metadata                | **VPC Flow Logs**                           |
| Packet inspection               | **Traffic Mirroring**                       |
| Private EC2 access              | **SSM Session Manager**                     |
| One VPC shared by accounts      | **VPC Sharing**                             |
| Dedicated hybrid connection     | **Direct Connect**                          |
| DX → VPC                        | **Private VIF**                             |
| DX → TGW                        | **Transit VIF + DX Gateway**                |
| Encrypted hybrid connection     | **Site-to-Site VPN**                        |
| DX backup                       | **VPN / another DX**                        |
| VPC Peering transit             | **No**                                      |
| Central transit routing         | **Transit Gateway**                         |
| Multiple remote VPN sites       | **VPN CloudHub**                            |
| Lambda + VPC + high concurrency | **ENI + subnet IP capacity**                |

---
