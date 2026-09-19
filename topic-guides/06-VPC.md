# Section 6: VPC — Virtual Private Cloud

## The idea

A **VPC (Virtual Private Cloud)** is your private network inside AWS.

You decide:

* which IP addresses the network uses
* where resources are placed
* which resources can access the internet
* which resources can communicate with each other
* how traffic is routed between resources and networks

The basic structure is:

```text
AWS Region
└── VPC
    ├── Subnet
    │   └── EC2
    └── Subnet
        └── Database
```

The most important idea:

> **VPC = your AWS network**

---

# Boxes in boxes

Everything is organized like this:

```text
AWS Region
└── VPC
    ├── Availability Zone A
    │   ├── Public Subnet
    │   │   └── EC2
    │   └── Private Subnet
    │       └── EC2
    │
    └── Availability Zone B
        ├── Public Subnet
        │   └── EC2
        └── Private Subnet
            └── Database
```

* A **VPC belongs to one Region**.
* A **subnet belongs to one Availability Zone**.
* Therefore, a multi-AZ architecture needs **multiple subnets**.
* A subnet contains resources such as EC2 instances.
* **Route tables** decide where network traffic goes.

Think of a route table as:

> **"Traffic going to X should go through Y."**

---

# Route tables

A route table contains routes that determine where traffic is sent.

Example:

```text
Destination       Target
0.0.0.0/0         Internet Gateway
10.0.0.0/16       local
```

The `local` route allows resources inside the VPC's CIDR range to communicate with each other according to security rules.

## Longest-prefix match

If multiple routes match the destination, AWS uses the **most specific route**.

This is called:

> **Longest-prefix match**

Example:

```text
10.0.0.0/16   → Transit Gateway
10.0.1.0/24   → VPC Peering
```

Traffic to:

```text
10.0.1.50
```

uses:

```text
10.0.1.0/24 → VPC Peering
```

because `/24` is more specific than `/16`.

### Memory

> **More specific route wins.**

---

# Public vs private: it's all about the route

A subnet is **public** when its route table has a route to an **Internet Gateway (IGW)**.

Example:

```text
0.0.0.0/0 → Internet Gateway
```

A subnet is **private** when it does not have a route to an Internet Gateway.

## Internet Gateway

An **Internet Gateway (IGW)** provides internet connectivity to the VPC.

```text
EC2
 ↓
IGW
 ↓
Internet
```

For IPv4, the IGW can support both:

```text
EC2 → Internet
Internet → EC2
```

assuming the required routing, public IP addressing, and security rules allow it.

### Most important rule

> **Public subnet = route table has a route to an IGW.**

Not the subnet name.

Not simply having a public IP.

**The route is what makes the subnet public.**

---

# Public subnet vs Internet-reachable EC2

A public subnet and a publicly reachable EC2 instance are related but not identical.

For an EC2 instance to be reachable from the IPv4 Internet, it normally needs:

```text
Public subnet
+
Public IPv4 address / Elastic IP
+
Security Group / NACL rules allowing the traffic
```

So:

```text
Public subnet
= route to IGW

Internet-reachable EC2
= public subnet
+ public IP
+ allowed traffic
```

A private instance can still have an IPv4 address inside the VPC, but without a public IPv4 address it is not directly reachable from the public Internet through the IGW.

---

# Internet access components

| VPC component                    | IP version  | Traffic       | Main purpose                                    |
| -------------------------------- | ----------- | ------------- | ----------------------------------------------- |
| **Internet Gateway (IGW)**       | IPv4 + IPv6 | Two-way       | Internet access for public resources            |
| **NAT Gateway**                  | IPv4        | Outbound only | Private IPv4 resources → Internet               |
| **Egress-Only Internet Gateway** | IPv6        | Outbound only | Private IPv6 resources → Internet               |
| **VPC Endpoint**                 | IPv4/IPv6   | Private       | Access AWS services without the public internet |

---

# NAT Gateway — outbound internet for private instances

A private EC2 instance may need to:

* download updates
* call an external API
* download packages

But you don't want the internet to start connections to that instance.

Use a **NAT Gateway**.

```text
Private EC2
     ↓
NAT Gateway
     ↓
Internet Gateway
     ↓
Internet
```

This allows:

```text
Private EC2 → Internet ✅
Internet → Private EC2 ❌
```

---

## NAT Gateway placement

A traditional **zonal NAT Gateway** is placed in a public subnet.

The private subnet's route table points to the NAT Gateway:

```text
0.0.0.0/0 → NAT Gateway
```

The NAT Gateway then uses the Internet Gateway to reach the internet.

---

## NAT Gateway availability

For the classic highly available architecture:

```text
AZ-A
Private Subnet
     ↓
NAT Gateway A

AZ-B
Private Subnet
     ↓
NAT Gateway B
```

This avoids depending on a single AZ's NAT Gateway.

### Why one NAT Gateway can be a problem

Suppose:

```text
AZ-A → NAT Gateway A
AZ-B → uses NAT Gateway A
```

If AZ-A fails, AZ-B can lose its internet path too.

For the traditional AZ-isolated design:

```text
AZ-A → NAT Gateway A
AZ-B → NAT Gateway B
```

### Important current AWS nuance

AWS also provides **Regional NAT Gateways**.

A Regional NAT Gateway is designed to automatically expand across Availability Zones, so you don't have to create separate zonal NAT gateways purely for AZ scaling.

For SAA questions, still remember the classic pattern:

> **One zonal NAT Gateway per AZ = traditional high-availability design**

But know that:

> **Regional NAT Gateway = managed cross-AZ expansion**

---

# NAT Gateway vs NAT Instance

### NAT Gateway

* AWS-managed
* automatically scales
* no security group
* pay for hourly usage + data processing

### NAT Instance

* EC2 instance that performs NAT
* you manage and patch it
* can have a security group
* must disable **source/destination checks**

---

# IPv6

NAT Gateway is mainly an **IPv4** solution.

For **IPv6 outbound-only internet access**, use:

> **Egress-Only Internet Gateway**

```text
IPv6 EC2
   ↓
Egress-Only Internet Gateway
   ↓
Internet
```

It allows:

```text
EC2 → Internet ✅
Internet → EC2 ❌
```

### Memory

> **IPv4 private outbound → NAT Gateway**

> **IPv6 private outbound-only → Egress-Only IGW**

---

# CIDR essentials

CIDR describes the IP range of your network.

Example:

```text
10.0.0.0/16
```

The `/16` tells you how much of the address is fixed.

Common examples:

```text
/16 → 65,536 addresses
/24 → 256 addresses
```

### Important facts

* VPC CIDR can be **/16 to /28**.
* AWS reserves **5 IPv4 addresses in every subnet**.
* Therefore, a `/24` subnet has:

```text
256 total
− 5 reserved
= 251 usable
```

---

# No overlapping CIDRs

Networks that need to connect should not have overlapping CIDRs.

Example:

```text
VPC-A: 10.0.0.0/16
VPC-B: 10.0.0.0/16
```

This causes problems for VPC connectivity such as:

* VPC Peering
* Transit Gateway routing
* VPN connectivity

Plan your CIDRs carefully.

### Important idea

> **Non-overlapping CIDRs make network routing and connectivity much easier.**

---

# The two guards: Security Groups vs NACLs

Both control network traffic, but they work differently.

|                           | Security Group (SG)              | Network ACL (NACL)      |
| ------------------------- | -------------------------------- | ----------------------- |
| Applies to                | **Network interface / instance** | **Subnet**              |
| Stateful?                 | **Yes**                          | **No**                  |
| Rules                     | **Allow only**                   | Allow + Deny            |
| Rule order                | All applicable rules             | **Lowest number first** |
| Can reference another SG? | **Yes**                          | No, uses CIDR           |

---

# Security Group

Security Groups are **stateful**.

Example:

```text
EC2 → Database
```

If the Security Group allows the connection out, the response is automatically allowed back.

You don't need to create a separate rule for the response.

### Example

Suppose an application server connects to a MySQL database:

```text
App Server SG
      ↓
Database SG
```

Database Security Group:

```text
Allow TCP 3306
Source: App Server SG
```

This is better than allowing a fixed IP range when the application instances may change IP addresses.

### Security Group references

A Security Group can reference another Security Group.

This is especially useful for architectures such as:

```text
ALB SG
 ↓
App SG
 ↓
Database SG
```

Each layer can allow traffic from the appropriate Security Group.

### Security Groups cannot DENY

Security Groups only have **allow rules**.

Therefore:

> **Block a specific IP → NACL**

---

# NACL

NACLs are **stateless**.

Every direction needs its own rule.

```text
Request:
Client → Server

Response:
Server → Client
```

Both directions must be allowed.

### Rule evaluation

NACL rules are evaluated from the **lowest rule number to the highest**.

The first matching rule is applied.

Example:

```text
Rule 100 → DENY 10.0.0.5/32
Rule 200 → ALLOW 0.0.0.0/0
```

Traffic from `10.0.0.5` matches rule 100 first and is denied.

---

# Ephemeral ports

When a server sends a response, it commonly uses an **ephemeral port**.

Typical range:

```text
1024–65535
```

Therefore, a NACL may need to allow these ports for return traffic.

### Exam trap

> **Request leaves, but response never comes back**

→ Check the **NACL**, especially ephemeral ports.

---

# Security Group vs NACL memory

```text
Security Group
= stateful
= instance / ENI
= allow only
= can reference SG

NACL
= stateless
= subnet
= allow + deny
= CIDR-based
= lowest rule number first
```

---

# VPC Endpoints — private access to AWS services

Normally, a private EC2 instance might use a NAT Gateway to access an AWS service.

VPC Endpoints provide a **private path to AWS services without using the public internet**.

There are two important endpoint types:

|                                      | Gateway Endpoint | Interface Endpoint       |
| ------------------------------------ | ---------------- | ------------------------ |
| Main services                        | **S3, DynamoDB** | Many AWS services + SaaS |
| Cost                                 | **Free**         | Paid                     |
| How it works                         | Route table      | ENI in your subnet       |
| Can be used from on-prem/peered VPC? | **No**           | **Yes**                  |

---

# Gateway Endpoint

Used for:

* S3
* DynamoDB

Example:

```text
Private EC2
    ↓
Gateway Endpoint
    ↓
S3
```

No NAT Gateway is required.

### Exam pattern

> "Private EC2 needs to access S3 privately and at the lowest cost."

→ **Gateway VPC Endpoint**

### Important

Gateway endpoints use route tables to provide private connectivity to supported AWS services.

---

# Interface Endpoint

An Interface Endpoint uses an **ENI** in your subnet.

It is based on **AWS PrivateLink**.

It can provide private access to many AWS services and SaaS services.

It can also be used from:

* VPC Peering
* Transit Gateway
* VPN
* on-premises networks

### Important exam distinction

> **S3/DynamoDB + normal VPC access + lowest cost → Gateway Endpoint**

> **Most other AWS services / SaaS / on-prem access → Interface Endpoint**

### Important current nuance

S3 and DynamoDB also support **Interface Endpoints**.

This matters when the architecture needs capabilities provided by an interface endpoint, such as private access from outside the VPC through supported connectivity.

So the better rule is:

```text
S3 / DynamoDB
+ normal VPC access
+ lowest cost
→ Gateway Endpoint

S3 / DynamoDB
+ need Interface Endpoint capabilities
→ Interface Endpoint
```

### Memory

> **Gateway = S3/DynamoDB and cheapest private VPC path**

> **Interface = ENI + PrivateLink + many services / advanced connectivity scenarios**

---

# Connecting VPCs: three very different tools

There are three concepts you must clearly separate:

```text
VPC Peering
= direct VPC-to-VPC connection

Transit Gateway
= central networking hub

PrivateLink
= access to one service
```

---

# VPC Peering — direct connection

VPC Peering connects **two VPCs directly**.

```text
VPC-A ←────────→ VPC-B
```

Important facts:

* Can be cross-account.
* Can be cross-region.
* **Not transitive.**
* CIDRs cannot overlap.

---

## Not transitive

If:

```text
VPC-A ↔ VPC-B
VPC-B ↔ VPC-C
```

That does **not** mean:

```text
VPC-A ↔ VPC-C
```

You would need another connection.

### Memory

> **VPC Peering = direct connection between two VPCs, not a transit network.**

---

# Transit Gateway (TGW) — central hub

Transit Gateway connects many networks through one central hub.

```text
             VPC-A
               |
VPC-B ---- Transit Gateway ---- VPC-C
               |
              VPN
               |
           On-premises
```

It can connect:

* VPCs
* Site-to-Site VPN
* Direct Connect

The big advantage:

> **Traffic can be transitive through the Transit Gateway.**

You don't need a separate connection between every pair of VPCs.

It can also be shared across AWS accounts using **AWS RAM**.

### When to use it

> **Many VPCs + centralized networking → Transit Gateway**

---

# PrivateLink — expose one service

PrivateLink is different.

It does **not** connect two complete networks.

Instead, it lets another VPC access **one specific service**.

```text
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

The customer gets access to that service, not the entire provider network.

### Important advantage

PrivateLink can work when the VPCs have **overlapping CIDRs** because it does not require normal network routing between the VPCs.

### When to use it

> **Give another VPC access to one service/API → PrivateLink**

Common example:

> A SaaS company wants thousands of customer VPCs to privately access its service.

→ **PrivateLink**

---

# Quick chooser

```text
2 VPCs need direct network communication
→ VPC Peering

Many VPCs / VPN / DX need centralized routing
→ Transit Gateway

Another VPC needs access to ONE service
→ PrivateLink
```

---

# Observability & admin access

# VPC Flow Logs

VPC Flow Logs record **network traffic metadata**.

They can show:

* source IP
* destination IP
* port
* protocol
* ACCEPT or REJECT

They do **not** capture the actual packet contents.

Use Flow Logs when asking:

> "Why is this connection being rejected?"

For example:

```text
ACCEPT
REJECT
REJECT
```

A REJECT can help identify a security rule or network-control problem.

### Memory

> **Flow Logs = metadata**

---

# Traffic Mirroring

Need the actual packets?

Use:

> **Traffic Mirroring**

Traffic Mirroring copies network traffic to a monitoring or inspection system.

### Memory

> **Traffic Mirroring = packets**

So:

```text
Flow Logs
= metadata

Traffic Mirroring
= packet copies
```

---

# Bastion Host

A bastion host is an EC2 instance used as a **jump server**.

```text
Your computer
      ↓ SSH
Bastion
      ↓ SSH
Private EC2
```

The bastion is normally in a public subnet.

Example Security Group setup:

```text
Bastion SG
→ SSH only from your trusted IP

Private EC2 SG
→ SSH from Bastion SG
```

You have to manage the bastion yourself.

---

# SSM Session Manager

SSM Session Manager is the modern alternative to a bastion host.

You can connect to private EC2 instances without:

* public IP
* open port 22
* bastion host

```text
You
 ↓
SSM Session Manager
 ↓
Private EC2
```

Sessions can also be logged.

### Exam pattern

> **"Most secure way to administer a private EC2 instance"**

→ **SSM Session Manager**

---

# VPC Sharing

VPC Sharing allows one AWS account to own the VPC while other accounts use its subnets.

```text
Network Account
      ↓
     VPC
   /     \
Account A  Account B
resources  resources
```

Use it when:

> **A central networking team owns the VPC, while multiple AWS accounts deploy applications into it.**

It uses **AWS RAM**.

---

# Hybrid Networking & Connectivity — SAA

## Core idea

Hybrid networking means:

> **Connect your on-premises network to AWS.**

The main services are:

* **Direct Connect (DX)** → dedicated private connection
* **Site-to-Site VPN** → encrypted connection over the Internet
* **VPC Peering** → connects two VPCs
* **Transit Gateway** → connects many networks
* **VPN CloudHub** → connects multiple remote networks through VPN

---

# Direct Connect and Virtual Interfaces (VIF)

Direct Connect has several important components:

```text
On-premises
    |
    | Direct Connect
    v
AWS
    |
    | VIF
    v
VPC / Transit Gateway
```

## Direct Connect connection

This is the **physical/dedicated network connection** between your on-premises network and AWS.

## Virtual Interface (VIF)

A VIF is a **logical connection** configured on top of Direct Connect.

It uses **BGP** to exchange routes.

---

# Private VIF

A **Private VIF** provides private connectivity to a VPC through a **Virtual Private Gateway (VGW)**.

```text
On-premises
     |
     | Direct Connect
     ↓
Private VIF
     ↓
VGW
     ↓
VPC
```

### Easy memory

> **Direct Connect = physical connection**

> **VIF = logical connection on top of it**

---

# Direct Connect Gateway

A **Direct Connect Gateway (DXGW)** allows Direct Connect connectivity to be used with supported AWS networking architectures across VPCs and Regions.

A common architecture is:

```text
On-premises
     |
Direct Connect
     |
   VIF
     |
 Direct Connect Gateway
     |
 ┌───┴───────────┐
 ↓               ↓
VPC / VGW      Transit Gateway
```

For architectures involving **Transit Gateway**, a **Transit VIF** is used to connect Direct Connect to the Transit Gateway through the Direct Connect Gateway.

### Memory

```text
Private VIF
→ VGW / VPC

Transit VIF
→ Direct Connect Gateway
→ Transit Gateway
```

### Exam signal

> **One Direct Connect architecture needs connectivity to multiple VPCs / a centralized Transit Gateway design**

→ Think **Direct Connect Gateway**

---

# Why the Private VIF targets the VPC's Region

A Private VIF connects to a **VGW**, and the VGW belongs to a specific VPC.

Example:

```text
On-premises
     |
     | Direct Connect
     ↓
Private VIF
     ↓
VGW
     ↓
VPC-1
```

The VIF must provide connectivity to the **correct VGW for the target VPC**.

### Exam memory

If the question says:

> "Create redundant Direct Connect connectivity to VPC-1"

make sure the new connection/VIF actually provides connectivity to **VPC-1**, not just to another VPC.

---

# Direct Connect redundancy

One Direct Connect connection can be a **single point of failure**.

```text
On-premises
     |
     | DX
     ↓
    VPC
```

If DX fails, the connection is lost.

For better fault tolerance, use another independent connection.

```text
             DX #1
On-premises -------> VPC
       \
        \ DX #2
         -------> VPC
```

Another option is to use VPN as a backup.

---

# VPN as a backup

```text
                 Direct Connect
On-premises ----------------------> VPC
     \
      \------ Internet VPN -------> VPC
```

If Direct Connect fails, the VPN can continue providing connectivity.

### Memory

> **DX + VPN = two independent paths**

---

# VPC Peering

VPC Peering connects two VPCs privately.

```text
VPC-1 ←────────→ VPC-2
```

But it is **not transitive**.

Example:

```text
VPC-1 ←→ VPC-2 ←→ VPC-3
```

does not mean:

```text
VPC-1 ←→ VPC-3
```

### Exam memory

> **VPC Peering = direct connection between two VPCs, not a transit network.**

---

# Exam question pattern

Suppose:

```text
On-premises
     |
Direct Connect
     |
   VPC-1
```

The question asks:

> "How can we make the connection to VPC-1 more fault tolerant?"

### Good answers

**1. Add a Site-to-Site VPN directly to VPC-1**

```text
On-premises
   | \
   |  \ VPN
   |   \
   DX   → VPC-1
```

**2. Add another independent Direct Connect connection to VPC-1**

```text
On-premises
   | \
 DX#1 DX#2
   |   |
   +---+
     ↓
   VPC-1
```

### Bad reasoning

If you add connectivity to VPC-2:

```text
On-premises
     ↓
   VPC-2
     ↓
  Peering
     ↓
   VPC-1
```

you should not assume that VPC-2 will act as a transit router.

**VPC Peering is not transitive.**

---

# Direct Connect vs VPN

| Feature    | Direct Connect                  | Site-to-Site VPN               |
| ---------- | ------------------------------- | ------------------------------ |
| Connection | Dedicated connection            | Internet                       |
| Encryption | Not automatically encrypted     | IPsec encrypted                |
| Main use   | Consistent private connectivity | Secure connectivity / backup   |
| Redundancy | Use multiple DX connections     | Use redundant VPN connectivity |

---

# VPC Peering vs Transit Gateway

|              | VPC Peering          | Transit Gateway    |
| ------------ | -------------------- | ------------------ |
| Connect VPCs | Yes                  | Yes                |
| Transitive   | **No**               | **Yes**            |
| Model        | Direct connection    | Central hub        |
| Best for     | Small number of VPCs | Many VPCs/networks |

---

# VPN CloudHub

**VPN CloudHub** allows multiple remote networks to communicate using a hub-and-spoke VPN architecture.

Conceptually:

```text
Branch A
    \
     \
      Transit / VPN hub
     /
    /
Branch B
```

It is useful when multiple sites need to communicate with each other through AWS VPN infrastructure.

### Memory

> **Multiple remote sites connected through VPN → VPN CloudHub**

---

# One-line mental model

```text
VPC
= AWS private network

IGW
= Internet connection

NAT Gateway
= Private IPv4 instances → Internet

Egress-Only IGW
= Private IPv6 instances → Internet, no inbound

Security Group
= Stateful firewall for instance / ENI

NACL
= Stateless firewall for subnet

Gateway Endpoint
= Private S3/DynamoDB access

Interface Endpoint
= Private access to AWS services / SaaS

VPC Peering
= Connect two VPCs

Transit Gateway
= Connect many networks

PrivateLink
= Give access to one service

Flow Logs
= Network traffic metadata

Traffic Mirroring
= Copy network packets

SSM Session Manager
= Secure access to private EC2
```

# Hybrid Networking — one-line mental model

```text
Direct Connect
= Dedicated connection to AWS

Direct Connect Gateway
= Centralize Direct Connect connectivity

Private VIF
= Direct Connect → VPC / VGW

Transit VIF
= Direct Connect → DX Gateway → Transit Gateway

Site-to-Site VPN
= Encrypted connection over Internet

DX + VPN
= Redundancy

VPC Peering
= Direct VPC-to-VPC connection

Transit Gateway
= Central hub for many networks

VPN CloudHub
= Connect multiple remote networks through VPN
```

---

# Question patterns

> **"Block all traffic from a specific IP address"**

→ **NACL DENY rule**

---

> **"Only app servers can connect to the database"**

→ **Database SG allows traffic from the app SG**

---

> **"Private EC2 needs private, free access to S3"**

→ **Gateway VPC Endpoint**

---

> **"25 VPCs and on-premises networks need to communicate"**

→ **Transit Gateway**

---

> **"Expose only one service to another VPC"**

→ **PrivateLink**

---

> **"CIDR ranges overlap but one service must be shared"**

→ **PrivateLink**

---

> **"Make NAT highly available across AZs"**

→ **One traditional zonal NAT Gateway per AZ**

---

> **"Private IPv4 EC2 needs Internet access"**

→ **NAT Gateway**

---

> **"IPv6 EC2 needs outbound Internet access but must reject unsolicited inbound connections"**

→ **Egress-Only Internet Gateway**

---

> **"Need to know whether traffic was accepted or rejected"**

→ **VPC Flow Logs**

---

> **"Need full packet information for inspection"**

→ **Traffic Mirroring**

---

> **"Most secure way to access private EC2"**

→ **SSM Session Manager**

---

> **"NAT instance is not forwarding traffic"**

→ **Disable source/destination check**

---

> **"Requests leave but responses do not return"**

→ **Check NACL rules for ephemeral ports**

---

> **"One Direct Connect connection must become fault tolerant"**

→ **Add another independent DX connection or a Site-to-Site VPN**

---

> **"Need connectivity from on-premises to a specific VPC"**

→ **Use Direct Connect/VPN connectivity that actually reaches that VPC**

---

> **"Need one Direct Connect architecture to reach multiple VPCs or a centralized Transit Gateway design"**

→ **Direct Connect Gateway**

---

> **"Multiple remote sites need to communicate using VPN through AWS"**

→ **VPN CloudHub**

---

> **"Multiple routes match a destination"**

→ **Longest-prefix match**

---

# Pocket card

| Keyword                           | Answer                                      |
| --------------------------------- | ------------------------------------------- |
| Public subnet                     | Route to IGW                                |
| Internet-reachable EC2            | Public subnet + public IP + allowed traffic |
| Private IPv4 → Internet           | NAT Gateway                                 |
| Traditional NAT high availability | One NAT Gateway per AZ                      |
| NAT Instance                      | Disable source/destination check            |
| IPv6 outbound-only                | Egress-Only IGW                             |
| Reserved IPs per subnet           | 5                                           |
| VPC CIDR                          | /16 to /28                                  |
| Longest matching route            | Most specific / longest prefix              |
| Block traffic                     | NACL DENY                                   |
| Stateful firewall                 | Security Group                              |
| Stateless firewall                | NACL                                        |
| SG-to-SG access                   | Reference another SG                        |
| S3/DynamoDB private access        | Gateway Endpoint                            |
| Other AWS services / SaaS         | Interface Endpoint                          |
| On-prem → private AWS service     | Interface Endpoint                          |
| Two VPCs                          | VPC Peering                                 |
| Many VPCs + VPN/DX                | Transit Gateway                             |
| One service to another VPC        | PrivateLink                                 |
| Traffic metadata                  | VPC Flow Logs                               |
| Full packet inspection            | Traffic Mirroring                           |
| Private EC2 administration        | SSM Session Manager                         |
| Multiple accounts, one VPC        | VPC Sharing                                 |
| Dedicated hybrid connection       | Direct Connect                              |
| DX → VPC / VGW                    | Private VIF                                 |
| DX → Transit Gateway              | Transit VIF + Direct Connect Gateway        |
| Encrypted Internet connection     | Site-to-Site VPN                            |
| DX backup                         | VPN or another DX                           |
| VPC Peering transit               | **No**                                      |
| Transit routing                   | Transit Gateway                             |
| Multiple remote VPN sites         | VPN CloudHub                                |
| AWS → one service                 | PrivateLink                                 |

---

# Final memory

```text
VPC
= AWS PRIVATE NETWORK

PUBLIC SUBNET
= ROUTE TO IGW

PRIVATE IPv4 OUTBOUND
= NAT GATEWAY

IPv6 OUTBOUND-ONLY
= EGRESS-ONLY IGW

SECURITY GROUP
= STATEFUL
= INSTANCE / ENI
= ALLOW ONLY

NACL
= STATELESS
= SUBNET
= ALLOW + DENY

GATEWAY ENDPOINT
= S3 / DYNAMODB
= LOWEST-COST PRIVATE VPC PATH

INTERFACE ENDPOINT
= ENI
= PRIVATElink
= MANY AWS SERVICES / SAAS

VPC PEERING
= TWO VPCS
= DIRECT
= NOT TRANSITIVE

TRANSIT GATEWAY
= MANY NETWORKS
= CENTRAL HUB
= TRANSITIVE

PRIVATELINK
= ONE SERVICE
= NOT WHOLE NETWORK

FLOW LOGS
= NETWORK METADATA

TRAFFIC MIRRORING
= PACKETS

SSM SESSION MANAGER
= SECURE PRIVATE EC2 ACCESS

DIRECT CONNECT
= DEDICATED CONNECTION

DIRECT CONNECT GATEWAY
= DX CENTRALIZATION

PRIVATE VIF
= DX → VGW / VPC

TRANSIT VIF
= DX → DXGW → TGW

SITE-TO-SITE VPN
= ENCRYPTED INTERNET CONNECTION

DX + VPN
= REDUNDANCY

VPN CLOUDHUB
= MULTIPLE REMOTE VPN NETWORKS

LONGEST-PREFIX MATCH
= MOST SPECIFIC ROUTE WINS
```

# The Golden Rule

```text
Public subnet
→ Route to IGW

Private IPv4 → Internet
→ NAT Gateway

IPv6 outbound-only
→ Egress-Only IGW

Stateful firewall
→ Security Group

Stateless firewall
→ NACL

Block an IP
→ NACL DENY

Private S3/DynamoDB
→ Gateway Endpoint

Private access to many AWS services / SaaS
→ Interface Endpoint

Two VPCs
→ VPC Peering

Many VPCs / centralized networking
→ Transit Gateway

One service to another VPC
→ PrivateLink

Traffic metadata
→ VPC Flow Logs

Actual packet inspection
→ Traffic Mirroring

Private EC2 administration
→ SSM Session Manager

Dedicated on-premises connection
→ Direct Connect

DX → VPC
→ Private VIF

DX → Transit Gateway
→ Transit VIF + Direct Connect Gateway

Encrypted backup path
→ Site-to-Site VPN

Multiple remote VPN sites
→ VPN CloudHub

Multiple matching routes
→ Longest-prefix match
```

> **Don't memorize the implementation.**
>
> **Memorize the unique signal.**

For example:

```text
Public subnet          → Route to IGW
Private IPv4 internet  → NAT Gateway
IPv6 outbound-only     → Egress-Only IGW
Stateful firewall      → Security Group
Stateless firewall     → NACL
S3/DynamoDB endpoint   → Gateway Endpoint
Many AWS services      → Interface Endpoint
Two VPCs               → VPC Peering
Many networks          → Transit Gateway
One service            → PrivateLink
Traffic metadata       → Flow Logs
Packet inspection      → Traffic Mirroring
Private EC2 access     → SSM Session Manager
Dedicated connection  → Direct Connect
DX → VPC               → Private VIF
DX → TGW               → Transit VIF + DXGW
VPN backup             → Site-to-Site VPN
Route conflict        → Longest-prefix match
```
