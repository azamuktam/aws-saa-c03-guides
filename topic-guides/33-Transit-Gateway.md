# Section 33: Transit Gateway

## The idea

**AWS Transit Gateway (TGW)** is a centralized network router for connecting multiple VPCs and other networks.

It is useful when an organization has:

* many VPCs
* multiple AWS accounts
* on-premises networks
* Site-to-Site VPN connections
* Direct Connect
* a need for **transitive routing**
* a need to centrally control network traffic

### VPC Peering vs Transit Gateway

**VPC Peering** connects two VPCs directly.

```text
VPC A ←→ VPC B
```

VPC Peering is **not transitive**.

```text
VPC A ←→ VPC B ←→ VPC C

VPC A ❌→ VPC C
```

If many VPCs need connectivity, the number of peering connections can grow rapidly.

With Transit Gateway:

```text
           Transit Gateway
          /       |       \
       VPC A    VPC B    VPC C
```

Each VPC connects to the Transit Gateway, and the Transit Gateway can route traffic between the attached networks.

### Easy rule

> **VPC Peering = direct VPC-to-VPC connection**

> **Transit Gateway = centralized hub for many networks**

---

# What can connect to Transit Gateway?

Transit Gateway supports several attachment types.

| Attachment                              | Purpose                                                  |
| --------------------------------------- | -------------------------------------------------------- |
| **VPC**                                 | Connect a VPC to the Transit Gateway                     |
| **Site-to-Site VPN**                    | Connect on-premises networks through VPN                 |
| **Direct Connect**                      | Connect through a Direct Connect Gateway and Transit VIF |
| **Transit Gateway peering**             | Connect two Transit Gateways                             |
| **Other supported network attachments** | Connect supported AWS/network resources                  |

### Typical architecture

```text
                       Transit Gateway
                    /        |        \
                   /         |         \
                VPC A      VPC B      VPC C
                   │
                VPN / DX
                   │
            On-premises network
```

---

# Transit Gateway routing

Transit Gateway uses **Transit Gateway route tables** to control how traffic is routed between attachments.

This allows centralized routing and segmentation.

For example:

```text
                 Transit Gateway
                /               \
           Prod route table    Dev route table
               /                    \
          Prod VPCs              Dev VPCs
```

You can configure routing so that:

* Production VPCs can communicate with on-premises
* Development VPCs can communicate with on-premises
* Production VPCs cannot route to development VPCs

### Important idea

> **Transit Gateway route tables can be used to segment traffic between groups of VPCs.**

---

# AWS RAM and Transit Gateway

**AWS Resource Access Manager (AWS RAM)** allows supported AWS resources to be **shared across AWS accounts**.

A major SAA use case is sharing a Transit Gateway with VPCs in other AWS accounts.

Example:

```text
Central networking account
          │
          │ creates
          ▼
    Transit Gateway
          │
          │ AWS RAM
      ┌───┴────┐
      ▼        ▼
 Account A   Account B
    VPC         VPC
```

This allows a central networking account to manage the Transit Gateway while other AWS accounts attach their VPCs to it.

### Important distinction

> **Transit Gateway = connects the networks**

> **AWS RAM = allows the Transit Gateway to be shared across accounts**

RAM does **not** perform the network routing itself.

### Exam clue

> "A central networking account has a Transit Gateway and wants VPCs in other AWS accounts to use it."

→ **Share the Transit Gateway using AWS RAM**

---

# Transit Gateway + Direct Connect

For on-premises connectivity through Direct Connect, the typical architecture is:

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
VPC A VPC B VPC C
```

The services have different roles:

| Service                    | Role                                              |
| -------------------------- | ------------------------------------------------- |
| **Direct Connect**         | Dedicated connectivity from on-premises to AWS    |
| **Direct Connect Gateway** | Connects Direct Connect to supported AWS networks |
| **Transit Gateway**        | Central routing hub for multiple VPCs/networks    |
| **AWS RAM**                | Shares the Transit Gateway across AWS accounts    |

### Exam clue

> "Existing Direct Connect + many VPCs/accounts need access to on-premises services."

→ **Direct Connect Gateway + Transit Gateway**

For multi-account architectures, the Transit Gateway can be shared using **AWS RAM**.

---

# Transit Gateway + Site-to-Site VPN

Transit Gateway can also be used as the central endpoint for multiple VPN connections.

```text
On-premises A
     │
    VPN
     │
     ▼
Transit Gateway
     ▲
     │
    VPN
     │
On-premises B
```

This is useful when several offices or on-premises networks need centralized connectivity to AWS.

---

# Transit Gateway Peering

You can connect two Transit Gateways using **Transit Gateway peering**.

Example:

```text
Region A                         Region B

VPC A ─┐                     ┌─ VPC D
VPC B ─┼─ TGW A ←→ TGW B ───┼─ VPC E
VPC C ─┘                     └─ VPC F
```

This is useful when different regions have separate Transit Gateways that need connectivity.

### Exam clue

> "Connect two Transit Gateways in different Regions."

→ **Transit Gateway peering**

---

# Appliance mode

**Appliance mode** is used when traffic must pass through a network appliance such as:

* firewall
* IDS/IPS
* inspection appliance
* security appliance

It helps maintain **symmetric traffic flows**, meaning both directions of a flow can pass through the same network appliance.

Example:

```text
VPC A
  │
  ▼
Transit Gateway
  │
  ▼
Inspection VPC
  │
Firewall
  │
  ▼
Destination
```

### Exam clue

> "All traffic must pass through the same firewall in both directions."

→ **Transit Gateway with appliance mode**

---

# Multicast

Transit Gateway supports **IP multicast**.

This is useful for applications that need one-to-many network communication.

### Exam clue

> "The application requires IP multicast."

→ **Transit Gateway**

For SAA, remember:

> **Multicast → Transit Gateway**

---

# Transit Gateway vs VPC Peering

| Feature                            | VPC Peering                                    | Transit Gateway                      |
| ---------------------------------- | ---------------------------------------------- | ------------------------------------ |
| Connects two VPCs directly         | ✅                                              | —                                    |
| Central hub                        | ❌                                              | ✅                                    |
| Transitive routing                 | ❌                                              | ✅                                    |
| Many VPCs                          | Possible, but many connections may be required | ✅                                    |
| Centralized routing                | ❌                                              | ✅                                    |
| Segmentation with TGW route tables | ❌                                              | ✅                                    |
| Cross-account architecture         | Possible                                       | ✅                                    |
| Cross-region connectivity          | Supported                                      | Supported through TGW peering        |
| Typical use                        | Small/simple VPC-to-VPC connectivity           | Large/multi-VPC network architecture |

### When should you prefer VPC Peering?

For a small number of VPCs that simply need direct communication, VPC Peering can be simpler and may have lower cost because it does not have Transit Gateway attachment/hourly charges.

### When should you prefer Transit Gateway?

Use Transit Gateway when you need:

* many VPCs
* centralized routing
* transitive connectivity
* centralized network segmentation
* multiple accounts
* VPN connectivity
* Direct Connect integration
* centralized inspection

---

# Transit Gateway vs PrivateLink

These solve very different problems.

## Transit Gateway

Provides **network-level connectivity**.

```text
VPC A
  │
  ▼
Transit Gateway
  │
  ▼
VPC B
```

The connected networks can route to each other according to the configured route tables.

## AWS PrivateLink

Provides **private access to a specific service**.

```text
Consumer VPC
     │
     ▼
Interface Endpoint
     │
     ▼
Service
```

The consumer does not receive general network access to the provider VPC.

### Easy rule

> **Need network-to-network connectivity → Transit Gateway**

> **Need access to one specific service → PrivateLink**

### PrivateLink exam clues

Use PrivateLink when the question emphasizes:

* expose a specific service
* many consumer VPCs
* consumers should not access the entire provider VPC
* overlapping CIDR ranges

---

# Cost considerations

Transit Gateway charges for:

* **Transit Gateway attachments**
* **data processed**

VPC Peering does not have a Transit Gateway-style hourly attachment charge, although data transfer charges still apply.

Therefore:

> **Small/simple VPC connectivity → VPC Peering may be simpler and cheaper**

> **Large/multi-VPC architecture → Transit Gateway provides centralized connectivity and routing**

Do not choose Transit Gateway only because it is more scalable; consider the actual number of VPCs and connectivity requirements.

---

# Question patterns

> **"25 VPCs and an on-premises datacenter need centralized connectivity."**

→ **Transit Gateway**

---

> **"VPC A peers with B, and B peers with C, but A cannot reach C."**

→ **VPC Peering is non-transitive**

If transitive connectivity is required:

→ **Transit Gateway**

---

> **"Production VPCs must not communicate with development VPCs, but both need access to on-premises."**

→ **Transit Gateway with separate route tables / routing policies**

---

> **"The application requires IP multicast."**

→ **Transit Gateway**

---

> **"Only two VPCs need simple direct communication."**

→ **VPC Peering**

---

> **"A central networking account needs to share a Transit Gateway with many AWS accounts."**

→ **AWS RAM**

---

> **"Connect Transit Gateway in Region A to Transit Gateway in Region B."**

→ **Transit Gateway peering**

---

> **"All traffic between VPCs must pass through a firewall appliance."**

→ **Transit Gateway with appliance mode**

---

> **"Existing Direct Connect must provide connectivity to many VPCs."**

→ **Direct Connect Gateway + Transit Gateway**

---

> **"Multiple AWS accounts need to use a central Transit Gateway."**

→ **AWS RAM + Transit Gateway**

---

> **"Expose one service to many customer VPCs without providing general VPC-to-VPC connectivity."**

→ **AWS PrivateLink**

---

# Pocket card

| Keyword                              | Answer                             |
| ------------------------------------ | ---------------------------------- |
| Many VPCs + central routing          | **Transit Gateway**                |
| Transitive routing                   | **Transit Gateway**                |
| VPC-to-VPC direct connection         | **VPC Peering**                    |
| VPC Peering is non-transitive        | **Yes**                            |
| Large number of VPCs                 | **Transit Gateway**                |
| Centralized network segmentation     | **TGW route tables**               |
| Share TGW across AWS accounts        | **AWS RAM**                        |
| Cross-region TGWs                    | **TGW peering**                    |
| DX → TGW                             | **Transit VIF + DX Gateway + TGW** |
| VPN → TGW                            | **Transit Gateway**                |
| Centralized firewall/inspection      | **TGW appliance mode**             |
| IP multicast                         | **Transit Gateway**                |
| Small/simple VPC-to-VPC connectivity | **VPC Peering**                    |
| Expose one service privately         | **PrivateLink**                    |

---

# Easy memory rules

```text
VPC Peering
→ Direct VPC-to-VPC connection
→ Non-transitive

Transit Gateway
→ Central network hub
→ Transitive routing
→ Many VPCs/networks

Transit Gateway Peering
→ Transit Gateway ↔ Transit Gateway

AWS RAM
→ Share supported resources across AWS accounts

Direct Connect Gateway
→ Connect Direct Connect to AWS network resources

PrivateLink
→ Private access to a specific service
```

## Multi-account architecture

```text
                       On-premises
                            │
                      Direct Connect
                            │
                   Direct Connect Gateway
                            │
                       Transit VIF
                            │
                            ▼
                     Transit Gateway
                            │
                     AWS RAM sharing
                    /        |        \
                   /         |         \
             Account A   Account B   Account C
                 │           │           │
                VPC         VPC         VPC
```

### Core SAA rules

> **VPC Peering → direct, non-transitive**

> **Transit Gateway → central hub, transitive**

> **AWS RAM → share Transit Gateway across accounts**

> **DX Gateway + Transit Gateway → many VPCs/accounts through Direct Connect**

> **PrivateLink → expose a specific service, not an entire network**
