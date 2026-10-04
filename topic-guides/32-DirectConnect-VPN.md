# Section 32: Direct Connect & VPN

## The big picture

AWS provides two main ways to connect an **on-premises network to AWS**:

* **Site-to-Site VPN** → encrypted IPsec connection over a network/internet path
* **Direct Connect (DX)** → dedicated network connection to AWS

The most important SAA skill is recognizing **which AWS-side architecture the question is describing**.

There are two especially important patterns:

```text
SITE-TO-SITE VPN

On-premises
     │
     │ IPsec VPN
     ▼
   ┌─────┐
   │ VGW │ ──→ VPC
   └─────┘
```

or:

```text
SITE-TO-SITE VPN + TRANSIT GATEWAY

On-premises
     │
     │ IPsec VPN
     ▼
Transit Gateway
   /    |    \
 VPC   VPC   VPC
```

Direct Connect uses a different path:

```text
DIRECT CONNECT + TRANSIT GATEWAY

On-premises
     │
 Direct Connect
     │
 Transit VIF
     │
     ▼
Direct Connect Gateway
     │
     ▼
Transit Gateway
   /    |    \
 VPC   VPC   VPC
```

### The key difference

> **VPN can attach directly to a Transit Gateway.**

> **Direct Connect reaches a Transit Gateway through a Transit VIF + Direct Connect Gateway.**

This distinction is extremely important for SAA questions.

---

# 1. Site-to-Site VPN

AWS Site-to-Site VPN creates an **IPsec VPN connection** between an on-premises network and AWS.

It normally uses an internet/network path rather than a dedicated Direct Connect circuit.

### Main characteristics

* Uses IPsec encryption
* Usually faster to deploy than Direct Connect
* Lower cost than dedicated connectivity
* Network performance and latency depend on the underlying network path
* Each VPN connection normally has **two tunnels** for redundancy
* Can connect to either a **Virtual Private Gateway** or a **Transit Gateway**

---

# 2. VPN Architecture: VGW vs Transit Gateway

This is the first major distinction to understand.

## A. VPN → Virtual Private Gateway → VPC

This is the traditional VPN architecture.

```text
On-premises
     │
     │ IPsec VPN
     ▼
Customer Gateway
     │
     ▼
Virtual Private Gateway
     │
     ▼
    VPC
```

### Components

| Component                         | Where            | Purpose                                   |
| --------------------------------- | ---------------- | ----------------------------------------- |
| **Customer Gateway (CGW)**        | On-premises side | Represents the customer's router/firewall |
| **Virtual Private Gateway (VGW)** | AWS              | VPN endpoint attached to a VPC            |

### Exam clue

> On-premises needs a VPN connection to **one VPC**.

Think:

**Site-to-Site VPN → VGW → VPC**

---

# 3. VPN → Transit Gateway → VPCs

A Site-to-Site VPN can also be attached **directly to a Transit Gateway**.

```text
On-premises
     │
     │ IPsec VPN
     ▼
Transit Gateway
   /    |    \
 VPC   VPC   VPC
```

There is **no Direct Connect Gateway** here.

### Why use this architecture?

Transit Gateway acts as a central router.

For example:

```text
                 Transit Gateway
                /       |       \
               /        |        \
             VPC A     VPC B     VPC C
               |
               |
        On-premises
             VPN
```

This is useful when the company has:

* Many VPCs
* Multiple AWS accounts
* A centralized networking architecture
* On-premises networks that need to communicate with many VPCs

### Exam clue

> On-premises needs VPN connectivity to **multiple VPCs**.

→ **Site-to-Site VPN → Transit Gateway**

> Cross-Region VPCs connect with each other**.
→ **Transit Gateway peering**

### Important

Do **not** think:

```text
VPN → VGW → TGW
```

for this architecture.

Instead:

```text
VPN → Transit Gateway
```

The VPN is itself a **Transit Gateway attachment**.

---

# 4. VPN: VGW vs Transit Gateway

| Requirement                          | Architecture                              |
| ------------------------------------ | ----------------------------------------- |
| VPN → one VPC                        | **VPN → VGW → VPC**                       |
| VPN → many VPCs                      | **VPN → TGW → VPCs**                      |
| Central hub for many VPCs            | **Transit Gateway**                       |
| Need higher aggregate VPN throughput | **TGW + ECMP + multiple VPN connections** |

### Easy memory

> **VGW = VPN to a VPC**

> **TGW = VPN to many VPCs**

---

# 5. Scaling VPN Throughput with ECMP

A common SAA requirement is:

> "Existing VPN connectivity does not provide enough bandwidth. Increase the VPN throughput."

Use:

**Transit Gateway + multiple VPN connections + ECMP**

```text
                 ┌── VPN Connection 1 ──┐
                 │                      │
On-premises ────┼── VPN Connection 2 ──┼── Transit Gateway
                 │                      │
                 └── VPN Connection 3 ─┘
```

ECMP (**Equal-Cost Multi-Path**) allows traffic to use multiple equal-cost paths.

### Important requirements

For Site-to-Site VPN ECMP:

* VPN connections must be attached to a **Transit Gateway**
* Use **dynamic routing / BGP**
* Multiple VPN connections can be used to increase aggregate throughput

```text
Multiple VPN connections
          +
    Transit Gateway
          +
      BGP / ECMP
          ↓
Higher aggregate VPN throughput
```

### Very important distinction

A single VPN connection already has **two tunnels**.

You do not simply create additional tunnels inside the same VPN connection to scale indefinitely.

For throughput scaling:

```text
❌ One VPN connection
   → add arbitrary extra tunnels

✅ Multiple VPN connections
   → Transit Gateway
   → ECMP
```

---

# 6. Large Bandwidth Tunnels

AWS also supports **Large Bandwidth Tunnels (LBT)** for supported Site-to-Site VPN configurations.

A Large Bandwidth Tunnel can support up to **5 Gbps per tunnel** in supported configurations.

The important exam distinction is:

```text
Standard VPN tunnel
→ up to approximately 1.25 Gbps

Large Bandwidth Tunnel
→ up to 5 Gbps per tunnel
```

Large Bandwidth Tunnels are supported for VPN connections attached to **Transit Gateway or Cloud WAN**, not Virtual Private Gateway attachments.

Both tunnels in a VPN connection must use the same bandwidth configuration.

### Need even more aggregate bandwidth?

Use:

```text
Multiple VPN connections
        ↓
Transit Gateway
        ↓
ECMP
```

For example:

```text
VPN 1
├── 5 Gbps
└── 5 Gbps

VPN 2
├── 5 Gbps
└── 5 Gbps

        ↓
      ECMP

Higher aggregate bandwidth
```

The exact achievable throughput depends on the architecture and configuration. For SAA questions, the important pattern is:

> **Need higher aggregate VPN throughput → Transit Gateway + ECMP + multiple VPN connections**

---

# 7. VPN Throughput vs VPN Redundancy

Do not confuse these requirements.

## Need higher throughput

```text
Transit Gateway
+
Multiple VPN connections
+
ECMP
+
BGP
```

## Need redundancy

```text
Multiple VPN paths
→ failover / resilience
```

A second Customer Gateway or another VPN path can improve resilience.

But when the question specifically says:

> **Increase aggregate VPN bandwidth**

look for:

**Transit Gateway + ECMP**

---

# 8. VPN CloudHub

**VPN CloudHub** is different from Transit Gateway.

VPN CloudHub allows **multiple on-premises sites to communicate with each other through AWS** using Site-to-Site VPN connections.

Example:

```text
Office A
    │
    │ VPN
    ▼
   VGW
   ▲  ▲
  /    \
VPN    VPN
/        \
Office B  Office C
```

The sites can communicate through AWS without creating a direct VPN connection between every pair.

### Example

```text
Office A ──┐
           │
Office B ──┼── AWS VPN CloudHub
           │
Office C ──┘
```

### Exam clue

> Several branch offices/data centers need to communicate with each other through AWS.

→ **VPN CloudHub**

---

# 9. VPN CloudHub vs Transit Gateway

These can look similar in exam questions, but the main idea is different.

|                        | VPN CloudHub                           | Transit Gateway                       |
| ---------------------- | -------------------------------------- | ------------------------------------- |
| Main purpose           | Connect multiple **on-premises sites** | Central hub for **VPCs and networks** |
| Main technology        | Site-to-Site VPN + VGW                 | Transit Gateway                       |
| Key clue               | Branch office ↔ branch office          | Many VPCs/networks                    |
| VPC connectivity       | Not the main purpose                   | Major use case                        |
| VPN throughput scaling | Not the main pattern                   | **ECMP**                              |

### Easy rule

> **Multiple on-premises sites need to communicate through AWS → VPN CloudHub**

> **Many VPCs/networks need a central routing hub → Transit Gateway**

---

# 10. Client VPN

**AWS Client VPN** is different from Site-to-Site VPN.

It provides secure access for **individual users/devices**.

```text
Employee laptop
       │
       │ Client VPN
       ▼
      AWS
```

### Remember

| Requirement                      | Service              |
| -------------------------------- | -------------------- |
| Entire on-premises network → AWS | **Site-to-Site VPN** |
| Individual laptop/user → AWS     | **Client VPN**       |

---

# 11. Direct Connect

**AWS Direct Connect (DX)** provides a dedicated network connection between an on-premises location and AWS.

```text
On-premises
     │
     │ Direct Connect
     ▼
    AWS
```

### Main characteristics

* Dedicated connectivity
* Does not use the public internet for the Direct Connect path
* More consistent network performance than internet-based VPN
* Suitable for large and steady data transfers
* Requires physical/network-provider connectivity
* Usually takes longer to provision than VPN
* **Does not encrypt traffic by default**

Common dedicated connection capacities include:

* 1 Gbps
* 10 Gbps
* 100 Gbps

---

# 12. Direct Connect Is Private, Not Encrypted

This is one of the most important exam traps.

> **Private connectivity does not automatically mean encryption.**

Direct Connect provides a private network path, but Direct Connect traffic is **not encrypted by default**.

If the requirement says:

> "Traffic must be encrypted."

You need an appropriate encryption mechanism, such as VPN/IPsec.

Remember:

```text
Direct Connect
= private connectivity

VPN / IPsec
= encryption
```

---

# 13. Direct Connect Virtual Interfaces

A **Virtual Interface (VIF)** determines how traffic is routed through a Direct Connect connection.

The three important VIF types are:

| VIF             | Main purpose                                                  |
| --------------- | ------------------------------------------------------------- |
| **Private VIF** | Private connectivity to VPC resources                         |
| **Public VIF**  | AWS public services                                           |
| **Transit VIF** | Transit Gateway connectivity through a Direct Connect Gateway |

---

# 14. Private VIF

A **Private VIF** is used for private connectivity to VPC resources.

Example:

```text
On-premises
     │
     │ Direct Connect
     ▼
Private VIF
     │
     ▼
    VPC
```

Typical clue:

> On-premises application needs private connectivity to EC2 instances in a VPC.

→ **Private VIF**

A private VIF can be used with a Virtual Private Gateway or a Direct Connect Gateway, depending on the architecture.

---

# 15. Public VIF

A **Public VIF** is used to access AWS public services through Direct Connect.

Examples:

* Amazon S3
* Amazon DynamoDB
* Other AWS public endpoints

Example:

```text
On-premises
     │
Direct Connect
     │
Public VIF
     │
     ▼
AWS public services
```

### Exam clue

> "On-premises needs Direct Connect access to S3."

→ **Public VIF**

---

# 16. Transit VIF

A **Transit VIF** is used when Direct Connect needs to reach a **Transit Gateway**.

The architecture is:

```text
On-premises
     │
     │ Direct Connect
     ▼
 Transit VIF
     │
     ▼
Direct Connect Gateway
     │
     ▼
Transit Gateway
   /    |    \
 VPC   VPC   VPC
```

### This is the critical Direct Connect architecture

You **do not** connect the Direct Connect connection directly to the Transit Gateway.

Instead:

```text
Direct Connect
      ↓
Transit VIF
      ↓
Direct Connect Gateway
      ↓
Transit Gateway
```

### Exam clue

> "Connect on-premises to many VPCs through an existing Direct Connect connection and Transit Gateway."

→ **Transit VIF + Direct Connect Gateway + Transit Gateway**

---

# 17. Why Does Direct Connect Need a Direct Connect Gateway?

This is the easiest way to remember it.

A Transit Gateway can accept several types of connectivity, including:

```text
VPC
VPN
Direct Connect Gateway
```

But the **Direct Connect connection itself is not a Transit Gateway attachment**.

Therefore the Direct Connect path is:

```text
Direct Connect connection
        ↓
    Transit VIF
        ↓
Direct Connect Gateway
        ↓
   Transit Gateway
```

By contrast, VPN can attach directly:

```text
Site-to-Site VPN
        ↓
   Transit Gateway
```

### Memorize this exact comparison

```text
VPN + TGW

VPN ─────────────→ TGW
```

```text
DX + TGW

DX
 ↓
Transit VIF
 ↓
DX Gateway
 ↓
TGW
```

This is one of the most useful architecture patterns in this section.

---

# 18. Direct Connect Gateway

A **Direct Connect Gateway (DXGW)** allows Direct Connect connectivity to be extended to AWS networks.

It is useful when an organization has:

* Multiple VPCs
* Multiple AWS accounts
* Multiple Regions
* Transit Gateway architecture

The architecture can be:

```text
On-premises
     │
Direct Connect
     │
Transit VIF
     │
     ▼
Direct Connect Gateway
     │
     ▼
AWS network
```

The DX Gateway can be associated with:

* **Transit Gateway**
* **Virtual Private Gateway(s)**

depending on the architecture.

---

# 19. Direct Connect Gateway + Transit Gateway

For a large multi-account environment:

```text
                    On-premises
                         │
                  Direct Connect
                         │
                    Transit VIF
                         │
                         ▼
               Direct Connect Gateway
                         │
                         ▼
                  Transit Gateway
                  /      |      \
                 /       |       \
             Account A Account B Account C
                 │         │         │
                VPC       VPC       VPC
```

This allows multiple VPCs/accounts to use the same Direct Connect connectivity to reach on-premises resources.

For example, the on-premises network contains:

* Active Directory
* DNS
* Internal applications

and several AWS accounts need access to them.

### Exam clue

> "Multiple AWS accounts need access to on-premises resources through an existing Direct Connect connection."

Think:

**DX → Transit VIF → DXGW → TGW → VPCs**

---

# 20. Direct Connect Gateway vs Transit Gateway

These services have different jobs.

| Service                    | Main job                                |
| -------------------------- | --------------------------------------- |
| **Direct Connect Gateway** | Connects Direct Connect to AWS networks |
| **Transit Gateway**        | Central router for VPCs and networks    |

Think of them as different layers:

```text
On-premises
     │
Direct Connect
     │
     ▼
DX Gateway
     │
     ▼
Transit Gateway
     │
 ┌───┼───┐
VPC  VPC  VPC
```

### Easy rule

> **DX Gateway = Direct Connect side**

> **Transit Gateway = AWS network routing side**

---

# 21. VPN + Transit Gateway vs DX + Transit Gateway

This is the most important comparison in this section.

## VPN

```text
On-premises
     │
     │ Internet / IPsec
     ▼
   VPN
     │
     ▼
Transit Gateway
   / | \
 VPC VPC VPC
```

**VPN attaches directly to the Transit Gateway.**

---

## Direct Connect

```text
On-premises
     │
     │ Direct Connect
     ▼
Transit VIF
     │
     ▼
DX Gateway
     │
     ▼
Transit Gateway
   / | \
 VPC VPC VPC
```

**Direct Connect reaches the Transit Gateway through the Direct Connect Gateway.**

### One-line memory

> **VPN → TGW directly**

> **DX → Transit VIF → DXGW → TGW**

---

# 22. Direct Connect vs VPC Peering

**VPC Peering** provides point-to-point connectivity between VPCs.

```text
VPC A ←────────→ VPC B
```

With many VPCs, this can become difficult to manage:

```text
VPC A ←→ VPC B
  ↕  ╲   ↗
VPC C ←→ VPC D
```

Transit Gateway provides a centralized architecture:

```text
          Transit Gateway
          /      |      \
        VPC A   VPC B   VPC C
```

### Exam clue

> "Many VPCs need centralized connectivity."

→ **Transit Gateway**

Not a large VPC peering mesh.

---

# 23. Multi-Account Transit Gateway

A Transit Gateway can be shared with other AWS accounts using **AWS Resource Access Manager (RAM)**.

Example:

```text
                 Transit Gateway
                /       |       \
               /        |        \
          Account A  Account B  Account C
              │          │          │
             VPC        VPC        VPC
```

This is common in organizations with a centralized networking account.

Combined with Direct Connect:

```text
On-premises
     │
Direct Connect
     │
Transit VIF
     │
DX Gateway
     │
Transit Gateway
     │
    RAM
     │
Multiple AWS accounts
```

---

# 24. Direct Connect + VPN Backup

A common exam requirement is:

> "The company already has Direct Connect and needs a cost-effective backup."

Use:

**Direct Connect as primary + Site-to-Site VPN as backup**

```text
                    ┌── Direct Connect ──→ AWS
On-premises ────────┤
                    └── Site-to-Site VPN → AWS
                              backup
```

The VPN provides an alternative path if Direct Connect becomes unavailable.

### Exam clue

> **"Cost-effective backup for Direct Connect"**

→ **Site-to-Site VPN**

---

# 25. Direct Connect + VPN Encryption

Direct Connect itself is private but does not encrypt traffic by default.

If encryption is required, VPN/IPsec can be used as an encryption layer.

Conceptually:

```text
On-premises
     │
     │ encrypted VPN traffic
     │
     ▼
Direct Connect
     │
     ▼
AWS
```

This is different from ordinary Site-to-Site VPN over the public internet.

### Important distinction

```text
Normal Site-to-Site VPN
→ VPN traffic uses an internet/network path
→ IPsec encryption
```

```text
VPN over Direct Connect / private IP VPN
→ VPN encryption runs over private Direct Connect connectivity
```

So if the question specifically says:

> **"Use Direct Connect but also encrypt the traffic."**

Think about a **VPN/IPsec encryption layer over the private Direct Connect path**, where supported.

---

# 26. Two Direct Connect Connections

For stronger Direct Connect resilience, an organization can use multiple Direct Connect connections.

```text
              ┌── DX connection 1 ──→ AWS
On-premises ──┤
              └── DX connection 2 ──→ AWS
```

Ideally, the connections should use independent physical/network paths where possible.

### Trade-off

```text
More resilience
      ↓
More infrastructure
      ↓
More cost
```

---

# 27. VPN First, Direct Connect Later

Direct Connect can take longer to provision.

A company can establish VPN connectivity first:

```text
Phase 1

On-premises
     │
     VPN
     │
     ▼
    AWS
```

Then add Direct Connect:

```text
Phase 2

On-premises
     │
     ├── Direct Connect ──→ AWS
     │
     └── VPN ─────────────→ AWS
```

The VPN can later become a backup path.

### Exam clue

> "The company needs connectivity immediately while waiting for Direct Connect."

→ **Site-to-Site VPN**

---

# 28. Choosing VPN vs Direct Connect

The decision usually comes down to these requirements:

| Requirement                          | Usually points to    |
| ------------------------------------ | -------------------- |
| Need connectivity quickly            | **Site-to-Site VPN** |
| Need IPsec encryption                | **Site-to-Site VPN** |
| Lower-cost connectivity              | **Site-to-Site VPN** |
| Dedicated physical connectivity      | **Direct Connect**   |
| More predictable network performance | **Direct Connect**   |
| Large, steady data transfers         | **Direct Connect**   |
| Cost-effective backup for DX         | **Site-to-Site VPN** |
| Individual users/laptops             | **Client VPN**       |

---

# 29. Architecture Decision Table

This is the table to use when solving SAA questions.

| Scenario                                       | Architecture                      |
| ---------------------------------------------- | --------------------------------- |
| On-prem → one VPC using VPN                    | **VPN → VGW → VPC**               |
| On-prem → many VPCs using VPN                  | **VPN → TGW → VPCs**              |
| VPN needs higher aggregate throughput          | **Multiple VPNs → TGW + ECMP**    |
| Multiple on-premises sites need to communicate | **VPN CloudHub**                  |
| Individual users need VPN access               | **Client VPN**                    |
| On-prem → one VPC using DX                     | **DX + Private VIF**              |
| On-prem → AWS public services using DX         | **DX + Public VIF**               |
| On-prem → many VPCs using DX + TGW             | **DX + Transit VIF + DXGW + TGW** |
| Multiple AWS accounts need DX connectivity     | **DXGW + TGW**                    |
| Need dedicated/private connectivity            | **Direct Connect**                |
| Need encryption over DX                        | **VPN/IPsec over DX**             |
| Need DX backup                                 | **DX + Site-to-Site VPN**         |
| Need central VPC/network hub                   | **Transit Gateway**               |
| Need point-to-point VPC connectivity           | **VPC Peering**                   |

---

# 30. The Four Most Important Architectures

If you remember only four diagrams, remember these.

## Architecture 1 — VPN to one VPC

```text
On-premises
     │
     │ VPN
     ▼
    VGW
     │
     ▼
    VPC
```

**Think: VPN + one VPC**

---

## Architecture 2 — VPN to many VPCs

```text
On-premises
     │
     │ VPN
     ▼
Transit Gateway
   /    |    \
 VPC   VPC   VPC
```

**Think: VPN + many VPCs**

---

## Architecture 3 — Direct Connect to one VPC

```text
On-premises
     │
Direct Connect
     │
Private VIF
     │
     ▼
    VPC
```

**Think: DX + Private VIF**

---

## Architecture 4 — Direct Connect to many VPCs through TGW

```text
On-premises
     │
Direct Connect
     │
Transit VIF
     │
DX Gateway
     │
Transit Gateway
   /    |    \
 VPC   VPC   VPC
```

**Think: DX + Transit VIF + DXGW + TGW**

---

# 31. The Critical Difference

When you see a question involving **Transit Gateway**, first ask:

> **Is the on-premises connection VPN or Direct Connect?**

Then choose the path.

### If VPN:

```text
VPN
 ↓
Transit Gateway
 ↓
VPCs
```

### If Direct Connect:

```text
Direct Connect
 ↓
Transit VIF
 ↓
Direct Connect Gateway
 ↓
Transit Gateway
 ↓
VPCs
```

### Never confuse these:

```text
❌ Direct Connect → Transit Gateway
```

```text
✅ Direct Connect → Transit VIF → DX Gateway → Transit Gateway
```

```text
✅ VPN → Transit Gateway
```

---

# 32. Exam Question Patterns

> **"The company needs connectivity to AWS within days."**

→ **Site-to-Site VPN**

---

> **"The company needs a dedicated connection with more consistent network performance for large, steady data transfers."**

→ **Direct Connect**

---

> **"On-premises needs VPN connectivity to several VPCs."**

→ **Site-to-Site VPN → Transit Gateway**

---

> **"On-premises needs connectivity to several VPCs using an existing Direct Connect connection."**

→ **Transit VIF → Direct Connect Gateway → Transit Gateway**

---

> **"VPN throughput is insufficient. Increase aggregate VPN bandwidth."**

→ **Multiple VPN connections + Transit Gateway + ECMP + BGP**

---

> **"Several branch offices need to communicate with each other through AWS."**

→ **VPN CloudHub**

---

> **"Remote employees need secure access to AWS from their laptops."**

→ **Client VPN**

---

> **"The company needs a cost-effective backup for Direct Connect."**

→ **Site-to-Site VPN**

---

> **"Direct Connect traffic must be encrypted."**

→ **VPN/IPsec over the Direct Connect connectivity**

Remember:

> **Direct Connect is private, but not encrypted by default.**

---

> **"On-premises needs access to S3 through Direct Connect."**

→ **Public VIF**

---

> **"On-premises needs private access to VPC resources through Direct Connect."**

→ **Private VIF**

---

> **"On-premises must reach VPCs attached to a Transit Gateway through Direct Connect."**

→ **Transit VIF + Direct Connect Gateway + Transit Gateway**

---

> **"Many AWS accounts need access to on-premises DNS and Active Directory through an existing Direct Connect connection."**

→ **Direct Connect Gateway + Transit Gateway**

---

> **"The company wants a central routing hub for many VPCs."**

→ **Transit Gateway**

---

> **"Two VPCs need simple point-to-point connectivity."**

→ **VPC Peering**

---

# 33. Pocket Card

## VPN

| Keyword                                 | Answer                                    |
| --------------------------------------- | ----------------------------------------- |
| Encrypted IPsec connection              | **Site-to-Site VPN**                      |
| Fast to deploy                          | **Site-to-Site VPN**                      |
| VPN → one VPC                           | **VPN → VGW → VPC**                       |
| VPN → many VPCs                         | **VPN → TGW → VPCs**                      |
| VPN throughput too low                  | **TGW + ECMP + multiple VPN connections** |
| ECMP VPN routing                        | **BGP / dynamic routing**                 |
| One VPN connection                      | **2 tunnels**                             |
| Multiple on-premises sites ↔ each other | **VPN CloudHub**                          |
| Individual users/laptops                | **Client VPN**                            |

---

## Direct Connect

| Keyword                        | Answer                       |
| ------------------------------ | ---------------------------- |
| Dedicated private connectivity | **Direct Connect**           |
| Consistent network performance | **Direct Connect**           |
| Large steady transfers         | **Direct Connect**           |
| DX encryption by default       | **❌ No**                     |
| Encrypt DX traffic             | **VPN/IPsec over DX**        |
| Private VPC resources          | **Private VIF**              |
| AWS public services            | **Public VIF**               |
| DX → Transit Gateway           | **Transit VIF + DXGW + TGW** |
| DX → many VPCs                 | **Transit VIF + DXGW + TGW** |
| DX backup                      | **Site-to-Site VPN**         |

---

## Transit Gateway

| Requirement               | Answer                            |
| ------------------------- | --------------------------------- |
| Central hub for many VPCs | **Transit Gateway**               |
| VPN → many VPCs           | **VPN → TGW**                     |
| DX → many VPCs            | **DX → Transit VIF → DXGW → TGW** |
| VPN throughput scaling    | **TGW + ECMP**                    |
| Multiple AWS accounts     | **TGW + AWS RAM**                 |

