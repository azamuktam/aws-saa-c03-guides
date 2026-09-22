# Section 32: Direct Connect & VPN

## The idea

Everything so far has lived *inside* AWS. But real companies have an office, or a whole datacenter, full of servers that need to talk to their VPCs (Virtual Private Clouds — your private networks in AWS). This section is about building that bridge.

You have two main ways to connect your building to AWS:

A **Site-to-Site VPN** (Virtual Private Network) is like sending an **armored car on public roads**: your data travels over the ordinary public internet, but wrapped in an encrypted IPsec tunnel so nobody can peek inside. It's fast to arrange and relatively cheap, but performance and latency depend on the internet.

**Direct Connect (DX)** is like building **your own private toll road** straight from your building to AWS: a dedicated network connection. It provides more consistent network performance and avoids the public internet, but physical connectivity takes longer to establish. ([docs.aws.amazon.com](https://docs.aws.amazon.com/directconnect/latest/UserGuide/Welcome.html?utm_source=chatgpt.com))

---

# Site-to-Site VPN

* **Encrypted IPsec tunnel over the public internet** by default.
* **Setup time: usually hours.**
* Relatively cheap compared with dedicated connectivity.
* Standard VPN tunnel bandwidth is up to **1.25 Gbps per tunnel**; Large Bandwidth Tunnels can support up to **5 Gbps per tunnel** when attached to a Transit Gateway or Cloud WAN. ([docs.aws.amazon.com](https://docs.aws.amazon.com/vpn/latest/s2svpn/VPNTunnels.html?utm_source=chatgpt.com))
* Latency varies because the traffic normally uses the public internet.

Two important components are involved:

| Component                         | Lives where | What it is                                  |
| --------------------------------- | ----------- | ------------------------------------------- |
| **Virtual Private Gateway (VGW)** | AWS side    | VPN endpoint attached to your VPC           |
| **Customer Gateway (CGW)**        | Your side   | Represents your on-premises router/firewall |

### Client VPN

**AWS Client VPN** is different from Site-to-Site VPN.

It connects **individual devices** to AWS rather than connecting an entire on-premises network.

Think:

> **Site-to-Site VPN = office/datacenter → AWS**

> **Client VPN = laptop/user → AWS**

---

# Direct Connect (DX)

* **Dedicated private network connection** from your location to AWS.
* Traffic does not traverse the public internet.
* Provides more consistent bandwidth and network performance.
* Common connection speeds include **1, 10, and 100 Gbps**, depending on the Direct Connect connection type.
* Setup can take **weeks to months** because physical connectivity may be required. ([docs.aws.amazon.com](https://docs.aws.amazon.com/directconnect/latest/UserGuide/Welcome.html?utm_source=chatgpt.com))

### Important trap

**Direct Connect is private, but it is NOT encrypted by default.**

Private connectivity ≠ encryption.

If the requirement is:

> "Traffic over Direct Connect must be encrypted."

Use an **encrypted VPN/IPsec overlay over the Direct Connect connectivity**, where supported.

### VPN over Direct Connect

Think:

```text
On-premises
     │
     │ Direct Connect
     ▼
AWS
     │
     └── Encrypted VPN/IPsec tunnel
```

So:

> **Direct Connect = private network path**

> **VPN = encryption**

---

# Direct Connect Virtual Interfaces (VIFs)

VIFs determine how traffic uses the Direct Connect connection.

| VIF type        | Used for                                                                      |
| --------------- | ----------------------------------------------------------------------------- |
| **Private VIF** | Accessing VPC resources using private IP addresses                            |
| **Public VIF**  | Accessing AWS public services such as S3 using public IP addresses            |
| **Transit VIF** | Accessing VPCs attached to a Transit Gateway through a Direct Connect Gateway |

([docs.aws.amazon.com](https://docs.aws.amazon.com/directconnect/latest/UserGuide/WorkingWithVirtualInterfaces.html?utm_source=chatgpt.com))

Think:

```text
Private VIF  → VPC/private resources
Public VIF   → AWS public services
Transit VIF  → Transit Gateway
```

---

# Direct Connect Gateway

A **Direct Connect Gateway (DXGW)** allows a Direct Connect connection to be used by multiple AWS networks.

It is especially important when the architecture contains:

* multiple VPCs
* multiple AWS accounts
* multiple Regions
* Transit Gateways

A Direct Connect Gateway can be associated with a **Transit Gateway**, allowing the Direct Connect connection to reach the VPCs attached to that Transit Gateway. ([docs.aws.amazon.com](https://docs.aws.amazon.com/directconnect/latest/UserGuide/direct-connect-transit-gateways.html?utm_source=chatgpt.com))

---

## Direct Connect Gateway + Transit Gateway

For a multi-account environment, use a **Direct Connect Gateway (DXGW) with a Transit Gateway (TGW)** to share one Direct Connect connection across multiple VPCs and AWS accounts.

Example:

```text
                 On-premises
              DNS + AD services
                     │
                     │ Direct Connect
                     ▼
             Direct Connect connection
                     │
                     │ Transit VIF
                     ▼
            Direct Connect Gateway
                     │
                     ▼
              Transit Gateway
             /        |        \
            /         |         \
         VPC-A      VPC-B      VPC-C
        Account A  Account B  Account C
```

The important idea is:

> **One Direct Connect connection can serve many VPCs instead of creating a separate DX connection for every VPC/account.**

The Transit Gateway provides the hub for the VPCs, while the Direct Connect Gateway connects the Direct Connect connection to the Transit Gateway. ([docs.aws.amazon.com](https://docs.aws.amazon.com/directconnect/latest/UserGuide/direct-connect-transit-gateways.html?utm_source=chatgpt.com))

### Across AWS accounts

The Direct Connect Gateway and Transit Gateway can be owned by **different AWS accounts**.

The Transit Gateway owner creates an association proposal, and the Direct Connect Gateway owner accepts it. This enables centralized connectivity in multi-account environments. ([docs.aws.amazon.com](https://docs.aws.amazon.com/directconnect/latest/UserGuide/direct-connect-transit-gateways.html?utm_source=chatgpt.com))

### Exam clue

> "The company already has Direct Connect and has multiple AWS accounts/VPCs that need consistent access to the same on-premises DNS and Active Directory services."

→ **Direct Connect Gateway + Transit Gateway**

### Why not a separate DX connection per account?

Creating another physical Direct Connect connection for every AWS account:

* costs more
* requires more infrastructure
* increases management overhead
* does not scale well

Instead:

```text
One DX connection
       ↓
Direct Connect Gateway
       ↓
Transit Gateway
       ↓
Many VPCs / AWS accounts
```

### Why not VPC peering?

VPC peering is primarily **point-to-point** connectivity.

With many VPCs and accounts, maintaining many peering relationships creates a complex mesh.

Transit Gateway is designed to act as a **central network hub**.

Think:

> **VPC Peering = point-to-point**

> **Transit Gateway = hub-and-spoke**

---

# Resiliency patterns

### DX + VPN failover

This is the classic exam pattern.

> "We need a **cost-effective backup** for our Direct Connect."

→ Add a **Site-to-Site VPN** as a backup path.

```text
                    ┌── Direct Connect ──→ AWS
On-premises ────────┤
                    └── VPN ────────────→ AWS
                         backup
```

The VPN provides a backup path if the Direct Connect connection becomes unavailable.

---

### Two Direct Connect connections

For stronger physical resiliency:

```text
On-premises
   │
   ├── DX connection 1
   │
   └── DX connection 2
```

Ideally, use different facilities/locations where appropriate.

Trade-off:

> Higher resiliency → higher cost.

---

### VPN now, DX later

If the company needs connectivity quickly but Direct Connect is still being provisioned:

```text
Today:
On-premises → VPN → AWS

Later:
On-premises → Direct Connect → AWS
```

This is a common migration pattern.

---

# Direct Connect vs Site-to-Site VPN

| Feature             | Site-to-Site VPN            | Direct Connect                     |
| ------------------- | --------------------------- | ---------------------------------- |
| Connection          | Public internet by default  | Dedicated private connection       |
| Encryption          | ✅ IPsec                     | ❌ Not encrypted by default         |
| Setup               | Fast                        | Slow                               |
| Cost                | Lower                       | Higher                             |
| Network performance | Variable                    | More consistent                    |
| Internet traversal  | Usually yes                 | No                                 |
| Best for            | Quick connectivity / backup | Large, steady, predictable traffic |
| Typical backup      | —                           | VPN is common backup               |

---

# The decision in one breath

* Need it in **hours**, **cheap**, **encrypted** → **Site-to-Site VPN**
* Need **consistent performance**, **large steady data volumes**, or **must avoid the public internet** → **Direct Connect**
* Want both reliability worlds → **DX primary + VPN backup**
* Need **many VPCs/accounts through one DX connection** → **Direct Connect Gateway + Transit Gateway**

---

# Question patterns

> **"Transferring 5 TB nightly; VPN performance is inconsistent."**

→ **Direct Connect**

Dedicated connectivity provides a more predictable network path for large, steady transfers.

---

> **"Cost-effective backup for an existing Direct Connect link."**

→ **Site-to-Site VPN**

---

> **"Data over Direct Connect must be encrypted in transit."**

→ **VPN/IPsec over the Direct Connect connectivity**

Remember:

> **DX is private, not encrypted by default.**

---

> **"Must connect on-premises to AWS within days."**

→ **Site-to-Site VPN**

Direct Connect generally takes longer to provision because dedicated connectivity is involved.

---

> **"Remote employees' laptops need secure access to the VPC."**

→ **AWS Client VPN**

---

> **"One Direct Connect connection must reach multiple VPCs."**

→ **Direct Connect Gateway**

For a Transit Gateway architecture:

→ **Direct Connect Gateway + Transit Gateway**

---

> **"One Direct Connect connection must serve VPCs in multiple AWS accounts."**

→ **Direct Connect Gateway + Transit Gateway**

---

> **"On-premises DNS and Active Directory services must be reachable from many AWS accounts using an existing DX connection."**

→ **Direct Connect Gateway + Transit Gateway**

---

> **"On-premises must reach AWS without traversing the public internet."**

→ **Direct Connect**

---

# Pocket card

| Keyword                                   | Answer                                         |
| ----------------------------------------- | ---------------------------------------------- |
| Encrypted tunnel, quick, relatively cheap | **Site-to-Site VPN**                           |
| VGW + CGW                                 | **Site-to-Site VPN components**                |
| Consistent bandwidth, dedicated           | **Direct Connect**                             |
| Setup in hours                            | **VPN**                                        |
| Setup in weeks–months                     | **Direct Connect**                             |
| Encrypt Direct Connect traffic            | **VPN/IPsec over DX**                          |
| Cost-effective DX backup                  | **Site-to-Site VPN**                           |
| One DX → many VPCs                        | **Direct Connect Gateway**                     |
| One DX → many VPCs/accounts via TGW       | **Direct Connect Gateway + Transit Gateway**   |
| Remote workers → VPC                      | **Client VPN**                                 |
| Private VIF                               | **Private VPC resources**                      |
| Public VIF                                | **AWS public services**                        |
| Transit VIF                               | **Transit Gateway via Direct Connect Gateway** |
| Many VPCs → central network hub           | **Transit Gateway**                            |
| Point-to-point VPC connectivity           | **VPC Peering**                                |

---

# Easy memory rules

```text
VPN
= quick + encrypted + internet

Direct Connect
= dedicated + private + consistent

DX Gateway
= share Direct Connect connectivity

Transit Gateway
= hub for many VPCs/accounts

Transit VIF
= Direct Connect → DX Gateway → Transit Gateway
```

### Multi-account DX pattern

```text
                 ON-PREMISES
              DNS / AD / Servers
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

**Remember:**

> **DX Gateway = connects Direct Connect to the AWS network**

> **Transit Gateway = connects many VPCs/accounts through a central hub**
