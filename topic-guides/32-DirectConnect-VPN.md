# Section 32: Direct Connect & VPN

## The idea

AWS provides several ways to connect an **on-premises network** to AWS.

The two main options are:

* **Site-to-Site VPN** → encrypted VPN connection over the internet
* **Direct Connect (DX)** → dedicated network connection between the on-premises location and AWS

The main exam decision is usually based on:

1. **How quickly must the connection be established?**
2. **Is encryption required?**
3. **How much traffic will be transferred?**
4. **Is consistent network performance required?**
5. **How many VPCs or AWS accounts need access?**
6. **Is the connection primary or backup connectivity?**

---

# Site-to-Site VPN

AWS Site-to-Site VPN creates an **encrypted IPsec connection** between an on-premises network and AWS over the internet.

### Main characteristics

* Uses the **public internet**
* Traffic is encrypted with **IPsec**
* Can usually be deployed much faster than Direct Connect
* Lower cost than dedicated connectivity
* Network performance and latency depend on the internet path
* Each VPN connection consists of **two tunnels** for redundancy/high availability.

### Main components

| Component                         | Location    | Purpose                                      |
| --------------------------------- | ----------- | -------------------------------------------- |
| **Virtual Private Gateway (VGW)** | AWS         | VPN endpoint attached to a VPC               |
| **Customer Gateway (CGW)**        | On-premises | Represents the customer's router or firewall |

### Basic architecture

```text
On-premises network
       │
       │ Internet
       ▼
Customer Gateway
       │
       │ IPsec VPN
       ▼
Virtual Private Gateway
       │
       ▼
      VPC
```

### Throughput

A standard Site-to-Site VPN tunnel supports up to approximately **1.25 Gbps**.

AWS also supports **Large Bandwidth Tunnels** with up to **5 Gbps per tunnel** in supported configurations. Large Bandwidth Tunnels are available for VPN connections attached to **Transit Gateway or Cloud WAN**, not Virtual Private Gateway attachments.

For SAA questions, the more important distinction is usually:

> **VPN = encrypted, fast to deploy, internet-based**

---

# Scaling VPN Throughput with ECMP

A common SAA requirement is:

> **"VPN connections are too slow during peak hours. Increase the VPN throughput."**

The solution is to use **multiple VPN connections with an AWS Transit Gateway and ECMP (Equal-Cost Multi-Path)**.

AWS explicitly supports ECMP for Site-to-Site VPN connections attached to a **Transit Gateway** to increase VPN bandwidth by aggregating multiple VPN tunnels. The VPN connections must use **dynamic routing/BGP**; ECMP is not supported for statically routed VPN connections.

```text
On-premises
     │
     ├── VPN Connection 1
     │     ├── Tunnel 1
     │     └── Tunnel 2
     │
     ├── VPN Connection 2
     │     ├── Tunnel 1
     │     └── Tunnel 2
     │
     └── VPN Connection 3
           ├── Tunnel 1
           └── Tunnel 2
                 │
                 ▼
          Transit Gateway
                 │
                VPCs
```

### Signal

> **Need higher VPN bandwidth / aggregate throughput → Transit Gateway + ECMP + multiple VPN connections**

---

## Why Transit Gateway?

ECMP for Site-to-Site VPN is supported on **Transit Gateway**.

It is **not** supported for VPN connections attached to a Virtual Private Gateway. AWS specifically recommends exploring ECMP when you want to use more than one VPN tunnel, because a Virtual Private Gateway does not support ECMP for Site-to-Site VPN.

```text
VGW
→ VPN connectivity
→ no ECMP aggregation

Transit Gateway
→ VPN connectivity
→ ECMP supported
```

---

## Important: more tunnels vs more VPN connections

A single Site-to-Site VPN connection already contains **two tunnels**. You do not simply turn one VPN connection into an arbitrary number of tunnels.

For throughput scaling:

```text
❌ One VPN connection
   → add more tunnels

✅ Multiple VPN connections
   → Transit Gateway
   → ECMP
   → aggregate the VPN tunnels
```

AWS documents that each VPN connection includes two tunnels, and ECMP can aggregate multiple VPN tunnels across VPN connections attached to a Transit Gateway.

---

## ECMP requires dynamic routing

For Site-to-Site VPN ECMP:

```text
VPN
+
Transit Gateway
+
BGP / dynamic routing
+
ECMP
```

Static routing does **not** support ECMP for Site-to-Site VPN.

### Memory

> **TGW + BGP + multiple VPN connections → ECMP**

---

## Example

Suppose the requirement is:

> "The company has multiple VPN connections, but employees experience slow connectivity during peak hours. Increase the VPN throughput."

Think:

```text
Multiple VPN connections
        ↓
Transit Gateway
        ↓
ECMP
        ↓
Use multiple VPN tunnels
        ↓
Higher aggregate VPN throughput
```

→ **Transit Gateway + ECMP**

---

## Large Bandwidth Tunnels

AWS now supports **Large Bandwidth Tunnels (LBT)** with up to **5 Gbps per tunnel** for supported Transit Gateway or Cloud WAN VPN connections. Both tunnels in a VPN connection must use the same bandwidth configuration.

```text
Standard tunnel
→ up to 1.25 Gbps

Large Bandwidth Tunnel
→ up to 5 Gbps
```

### Scaling beyond 5 Gbps

AWS specifically documents using **ECMP across multiple VPN connections** when bandwidth requirements exceed 5 Gbps per tunnel.

For example:

```text
VPN Connection 1
├── 5 Gbps
└── 5 Gbps

VPN Connection 2
├── 5 Gbps
└── 5 Gbps

        ↓
      ECMP
        ↓
    20 Gbps
```

This is an example of aggregating multiple VPN tunnels using ECMP.

For SAA questions, however, the key concept is:

> **Need higher aggregate VPN throughput → multiple VPN connections + Transit Gateway + ECMP**

---

## Throughput vs Redundancy

Do not confuse these two requirements.

### Need higher throughput

```text
Transit Gateway
+
ECMP
+
multiple VPN connections
```

### Need redundancy

```text
Multiple VPN paths / customer gateways
→ failover
```

A second Customer Gateway can improve **resilience**, but the SAA pattern for **scaling aggregate VPN throughput** is:

> **Transit Gateway + ECMP + multiple VPN connections**

---

# Client VPN

**AWS Client VPN** is different from Site-to-Site VPN.

It provides secure access for **individual users/devices**.

| Requirement                      | Service              |
| -------------------------------- | -------------------- |
| Entire on-premises network → AWS | **Site-to-Site VPN** |
| Individual laptop/user → AWS     | **Client VPN**       |

Example:

```text
Employee laptop
      │
      │ Client VPN
      ▼
     AWS
```

---

# Direct Connect

**AWS Direct Connect (DX)** provides a dedicated network connection between an on-premises location and AWS.

### Main characteristics

* Does **not** use the public internet for the Direct Connect path
* Provides more consistent network performance than internet-based VPN
* Suitable for large and steady data transfers
* Requires physical/network-provider connectivity
* Usually takes longer to provision than a VPN
* **Does not encrypt traffic by default**

Common Direct Connect speeds depend on the connection type and location. AWS Direct Connect supports multiple connection capacities, including dedicated connections such as 1 Gbps, 10 Gbps, and 100 Gbps.

### Basic architecture

```text
On-premises network
       │
       │ Direct Connect
       ▼
AWS
       │
       ▼
     VPC
```

---

# Direct Connect is private, not encrypted

This is one of the most important exam traps.

> **Private connectivity does not automatically mean encrypted traffic.**

Direct Connect traffic is private, but Direct Connect does not provide encryption by default.

If the requirement is:

> "Traffic must be encrypted in transit."

Use an appropriate **VPN/IPsec encryption layer over the Direct Connect connectivity**.

### Remember

```text
Direct Connect
= private connectivity

VPN/IPsec
= encryption
```

---

# Direct Connect Virtual Interfaces (VIFs)

A **Virtual Interface (VIF)** determines how traffic is routed through the Direct Connect connection.

There are three important VIF types:

| VIF             | Used for                                                   |
| --------------- | ---------------------------------------------------------- |
| **Private VIF** | Access to VPC resources using private IP addresses         |
| **Public VIF**  | Access to AWS public services using public IP addresses    |
| **Transit VIF** | Access to Transit Gateway through a Direct Connect Gateway |

AWS documents these three VIF types and their purposes.

---

### Private VIF

Used when the connection is directly to a VPC through a **Virtual Private Gateway** or through a **Direct Connect Gateway**, depending on the architecture.

Example:

```text
On-premises
     │
     │ DX
     ▼
Private VIF
     │
     ▼
VPC
```

Typical use:

> On-premises application needs private access to EC2 instances in a VPC.

---

### Public VIF

Used to access AWS public services.

Examples include:

* Amazon S3
* Amazon DynamoDB

The traffic uses AWS public service endpoints rather than a private VPC address.

Typical clue:

> "On-premises needs Direct Connect access to S3."

→ **Public VIF**

---

### Transit VIF

Used when Direct Connect needs to connect to a **Transit Gateway**.

The usual architecture is:

```text
On-premises
     │
Direct Connect
     │
Transit VIF
     │
Direct Connect Gateway
     │
Transit Gateway
     │
 ┌───┼────┐
VPC  VPC  VPC
```

Typical clue:

> "Connect on-premises to many VPCs through a Transit Gateway."

→ **Transit VIF + Direct Connect Gateway + Transit Gateway**

AWS documents that a transit VIF is used to access one or more Transit Gateways associated with Direct Connect gateways.

---

# Direct Connect Gateway

A **Direct Connect Gateway (DXGW)** is used to extend a Direct Connect connection to multiple AWS networks.

It is especially useful when an organization has:

* multiple VPCs
* multiple AWS accounts
* multiple AWS Regions
* a Transit Gateway architecture

Without a DX Gateway, creating separate Direct Connect connectivity for every VPC would create unnecessary complexity.

### Basic idea

```text
On-premises
     │
Direct Connect
     │
     ▼
Direct Connect Gateway
     │
   ┌─┴───────┐
   ▼         ▼
  VPC       VPC
```

AWS supports associating a Direct Connect gateway with a **Transit Gateway** or with multiple **Virtual Private Gateways**, depending on the architecture.

---

# Direct Connect Gateway + Transit Gateway

For a large multi-account environment, a common architecture is:

```text
                  On-premises
                DNS / AD / Apps
                     │
                     │
              Direct Connect
                     │
                     ▼
            Direct Connect Gateway
                     │
                Transit VIF
                     │
                     ▼
              Transit Gateway
              /      |      \
             /       |       \
         Account A Account B Account C
            │         │         │
           VPC       VPC       VPC
```

This allows multiple VPCs and AWS accounts to use the same Direct Connect connectivity to reach on-premises services.

AWS documents the Transit VIF → Direct Connect Gateway → Transit Gateway path for accessing VPCs or VPNs attached to the Transit Gateway.

---

## Example

Suppose the company has:

```text
On-premises:
- DNS
- Active Directory
- Internal applications

AWS:
- Account A → VPC A
- Account B → VPC B
- Account C → VPC C
```

The requirement is:

> All AWS accounts must have dedicated connectivity to the on-premises DNS and Active Directory services.

A scalable architecture is:

```text
On-premises
     │
Direct Connect
     │
DX Gateway
     │
Transit Gateway
   /   |   \
 VPC  VPC  VPC
```

### Why this is useful

You avoid:

* creating a separate Direct Connect connection for each account
* maintaining many independent connections
* unnecessary physical connectivity costs

### Exam clue

> "The company already has Direct Connect and multiple AWS accounts need consistent access to on-premises DNS and Active Directory."

→ **Direct Connect Gateway + Transit Gateway**

---

# Direct Connect Gateway vs Transit Gateway

These services solve different problems.

| Service                    | Main purpose                                       |
| -------------------------- | -------------------------------------------------- |
| **Direct Connect Gateway** | Connect Direct Connect to multiple AWS networks    |
| **Transit Gateway**        | Central routing hub for multiple VPCs and networks |

Think:

```text
Direct Connect
      │
      ▼
DX Gateway
      │
      ▼
Transit Gateway
      │
 ┌────┼────┐
VPC  VPC  VPC
```

### Easy rule

> **DX Gateway = Direct Connect connectivity**

> **Transit Gateway = VPC/network connectivity**

---

# Direct Connect Gateway vs VPC Peering

**VPC Peering** provides point-to-point connectivity between VPCs.

For example:

```text
VPC A ←→ VPC B
```

If there are many VPCs, you would need many separate peering relationships.

A Transit Gateway provides a centralized architecture:

```text
          Transit Gateway
          /      |      \
        VPC A   VPC B   VPC C
```

Therefore:

> **Many VPCs → Transit Gateway**

rather than creating a large VPC peering mesh.

---

# Multi-account access

The Transit Gateway can be shared with other AWS accounts using **AWS Resource Access Manager (RAM)**.

Example:

```text
                    Transit Gateway
                    /      |      \
                   /       |       \
              Account A Account B Account C
                  │         │         │
                 VPC       VPC       VPC
```

This allows a central networking account to provide connectivity to VPCs owned by other accounts.

For a multi-account Direct Connect design:

```text
On-premises
     │
Direct Connect
     │
DX Gateway
     │
Transit Gateway
     │
AWS RAM
     │
Multiple AWS accounts
```

---

# Resiliency patterns

## Direct Connect + VPN backup

A common exam pattern is:

> "We already have Direct Connect and need a cost-effective backup."

Use:

**Direct Connect as primary + Site-to-Site VPN as backup**

```text
                   ┌── Direct Connect ──→ AWS
On-premises ───────┤
                   └── Site-to-Site VPN → AWS
                         backup
```

The VPN provides an alternative path if Direct Connect becomes unavailable.

### Exam clue

> **"Cost-effective backup for Direct Connect"**

→ **Site-to-Site VPN**

---

# Two Direct Connect connections

For stronger connectivity resilience, an organization can use multiple Direct Connect connections.

For example:

```text
              ┌── DX connection 1 ──→ AWS
On-premises ──┤
              └── DX connection 2 ──→ AWS
```

Ideally, the connections should use independent physical/network paths when possible.

Trade-off:

> More resilience → more cost and infrastructure.

---

# VPN first, Direct Connect later

A company may need connectivity immediately while waiting for Direct Connect provisioning.

A common migration approach is:

```text
Phase 1
On-premises
     │
     VPN
     │
     ▼
    AWS


Phase 2
On-premises
     │
Direct Connect
     │
     ▼
    AWS
```

This allows the organization to establish connectivity quickly and move to dedicated connectivity later.

---

# Direct Connect vs Site-to-Site VPN

| Feature                             | Site-to-Site VPN   | Direct Connect   |
| ----------------------------------- | ------------------ | ---------------- |
| Uses public internet                | ✅                  | ❌                |
| Encryption                          | ✅ IPsec            | ❌ Not by default |
| Deployment speed                    | Fast               | Slower           |
| Cost                                | Lower              | Higher           |
| Performance                         | Internet-dependent | More consistent  |
| Suitable for large steady transfers | Sometimes          | ✅                |
| Private AWS connectivity            | ✅                  | ✅                |
| Backup for DX                       | ✅                  | —                |
| Dedicated physical connection       | ❌                  | ✅                |

### Decision rule

| Requirement                                           | Choose                           |
| ----------------------------------------------------- | -------------------------------- |
| Need connectivity quickly                             | **Site-to-Site VPN**             |
| Need encrypted connectivity                           | **Site-to-Site VPN**             |
| Need dedicated/private connectivity                   | **Direct Connect**               |
| Large, steady data transfers                          | **Direct Connect**               |
| More predictable network performance                  | **Direct Connect**               |
| Cost-effective backup for DX                          | **Site-to-Site VPN**             |
| Need many VPCs through one DX connection              | **Direct Connect Gateway**       |
| Need many VPCs/accounts through a central network hub | **DX Gateway + Transit Gateway** |
| Individual users need secure AWS access               | **Client VPN**                   |

---

# VPN Throughput vs Direct Connect

A useful distinction:

```text
Need higher VPN throughput
→ Transit Gateway + ECMP + multiple VPN connections

Need dedicated high-capacity connectivity
→ Direct Connect
```

Do not automatically choose Direct Connect just because VPN throughput is insufficient.

If the question specifically asks to **scale existing Site-to-Site VPN throughput**, look for:

> **Transit Gateway + ECMP**

---

# Question patterns

> **"Transferring 5 TB nightly and VPN performance is inconsistent."**

→ **Direct Connect**

Large and predictable data transfers are a common use case for dedicated connectivity.

---

> **"VPN connections are slow during peak hours. Increase VPN throughput."**

→ **Transit Gateway + ECMP + additional VPN connections**

The goal is to aggregate multiple VPN tunnels for higher bandwidth.

---

> **"We need a cost-effective backup for an existing Direct Connect connection."**

→ **Site-to-Site VPN**

---

> **"Traffic sent through Direct Connect must be encrypted."**

→ **Use VPN/IPsec encryption over the Direct Connect connectivity**

Remember:

> **Direct Connect is private, but not encrypted by default.**

---

> **"The company must connect on-premises to AWS within days."**

→ **Site-to-Site VPN**

Direct Connect normally takes longer to provision.

---

> **"Remote employees' laptops need secure access to AWS."**

→ **AWS Client VPN**

---

> **"One Direct Connect connection must reach multiple VPCs."**

→ **Direct Connect Gateway**

---

> **"Multiple AWS accounts need access to on-premises DNS and Active Directory through an existing Direct Connect connection."**

→ **Direct Connect Gateway + Transit Gateway**

---

> **"The company wants a central networking hub for many VPCs."**

→ **Transit Gateway**

---

> **"The same Direct Connect connection must provide connectivity to VPCs attached to a Transit Gateway."**

→ **Transit VIF + Direct Connect Gateway + Transit Gateway**

---

> **"On-premises needs access to S3 through Direct Connect."**

→ **Public VIF**

---

> **"On-premises needs private connectivity to a VPC."**

→ **Private VIF**

---

> **"On-premises VPN connections must use more than one tunnel simultaneously to increase bandwidth."**

→ **Transit Gateway + ECMP**

---

# Pocket card

| Keyword                                | Answer                              |
| -------------------------------------- | ----------------------------------- |
| Encrypted tunnel over internet         | **Site-to-Site VPN**                |
| Fast to deploy                         | **Site-to-Site VPN**                |
| VGW + CGW                              | **Site-to-Site VPN**                |
| Dedicated private connection           | **Direct Connect**                  |
| Consistent network performance         | **Direct Connect**                  |
| Large steady transfers                 | **Direct Connect**                  |
| Direct Connect is encrypted by default | **❌ No**                            |
| Encrypt Direct Connect traffic         | **VPN/IPsec over DX**               |
| Cost-effective DX backup               | **Site-to-Site VPN**                |
| Individual users/laptops → AWS         | **Client VPN**                      |
| One DX → multiple VPCs                 | **Direct Connect Gateway**          |
| One DX → many VPCs through TGW         | **DX Gateway + Transit Gateway**    |
| Central hub for many VPCs              | **Transit Gateway**                 |
| Private VIF                            | **Private VPC resources**           |
| Public VIF                             | **AWS public services**             |
| Transit VIF                            | **Transit Gateway**                 |
| Many AWS accounts need on-prem access  | **DX Gateway + Transit Gateway**    |
| Point-to-point VPC connectivity        | **VPC Peering**                     |
| VPN throughput too low                 | **Transit Gateway + ECMP**          |
| Aggregate multiple VPN tunnels         | **ECMP**                            |
| ECMP VPN routing                       | **BGP / dynamic routing**           |
| One VPN connection                     | **2 tunnels**                       |
| Scale VPN throughput                   | **Multiple VPN connections + ECMP** |

---
