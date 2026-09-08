# Section 6: VPC — Virtual Private Cloud

## The idea

A VPC is your own **private, fenced-off patch of the AWS cloud** — a network that belongs entirely to you. Think of it as buying a plot of land in a huge city: you decide the street layout (IP ranges), you build rooms (subnets), you decide which rooms have doors to the street (internet access), and you hire guards to check who comes and goes (security groups and NACLs). Nothing gets in or out unless *you* built a door for it.

This is the **biggest topic on the SAA-C03 exam** — networking questions are everywhere, and they're very pattern-matchable once the mental model clicks. So let's build it.

## Boxes in boxes

Everything in VPC-land nests:

```
┌──────────────── AWS Region (e.g., us-east-1) ────────────────┐
│  ┌──────────────── VPC  10.0.0.0/16 ────────────────────┐    │
│  │  ┌──── AZ-a ──────────────┐  ┌──── AZ-b ───────────┐ │    │
│  │  │ Public subnet          │  │ Public subnet       │ │    │
│  │  │  10.0.1.0/24           │  │  10.0.2.0/24        │ │    │
│  │  │  [web EC2]  [NAT GW]   │  │  [web EC2] [NAT GW] │ │    │
│  │  ├────────────────────────┤  ├─────────────────────┤ │    │
│  │  │ Private subnet         │  │ Private subnet      │ │    │
│  │  │  10.0.11.0/24          │  │  10.0.12.0/24       │ │    │
│  │  │  [app EC2]  [database] │  │  [app EC2] [db]     │ │    │
│  │  └────────────────────────┘  └─────────────────────┘ │    │
│  │            ▲ Internet Gateway (IGW) — the front door  │    │
│  └────────────┼──────────────────────────────────────────┘    │
└───────────────┼───────────────────────────────────────────────┘
                ▼  Internet
```

- **Region > VPC > subnets > instances.** A VPC lives in ONE region; a **subnet lives in ONE Availability Zone** (so multi-AZ = multiple subnets).
- **Route tables are the road signs.** Each subnet has a route table saying "traffic for X goes via Y." Change the signs, change the network.

## Public vs private: it's all about the route

- An **Internet Gateway (IGW)** is the VPC's **two-way front door** — traffic can go out AND the internet can come in (to public IPs).
- **THE definition** — memorize this exactly: a subnet is **public** if its route table has a route to the IGW (`0.0.0.0/0 → igw-xxx`). No IGW route = **private**. That's it. Not the name, not the IPs — the route.

### NAT Gateway — the outbound-only door

Private instances still need to download patches and call external APIs. Enter the **NAT Gateway** (Network Address Translation). Picture a **receptionist who mails letters for everyone in the building**: she takes your letter, **swaps the return address for the building's front-desk address**, and **remembers who asked** — so when the reply comes back, she hands it to the right person. But a stranger can't walk up and mail a letter *IN* to you; if nobody inside asked for it, she bins it.

That's NAT: **private instances can initiate outbound to the internet; the internet can never initiate inbound.**

Lock in these facts:

- The NAT Gateway **lives IN a public subnet** — it needs the IGW route itself to reach the internet. Private subnets then route `0.0.0.0/0 → nat-xxx`.
- A NAT Gateway is **per-AZ**. THE trap: one NAT Gateway shared by all AZs = a hidden single point of failure — if its AZ dies, every private subnet loses internet. *"Make NAT highly available"* → **one NAT Gateway per AZ**, each private subnet routing to its local one.
- NAT Gateway is **managed, scales automatically (up to ~45–100 Gbps), no security groups on it**, pay per hour + per GB.
- **NAT Instance** = the legacy DIY version: an EC2 instance doing NAT. It CAN have a security group, you manage/patch/size it yourself, and — THE NAT-instance answer — you must **disable the source/destination check** on it (EC2 normally drops traffic not addressed to itself; a NAT forwards other people's traffic, so the check must go).
- IPv6 has no NAT (every IPv6 address is public). The IPv6 equivalent of "outbound-only" is the **egress-only Internet Gateway**. *"IPv6 instances need outbound internet but no inbound"* → egress-only IGW.

## CIDR essentials

CIDR (Classless Inter-Domain Routing) notation like `10.0.0.0/16` means "the first 16 bits are fixed; the rest are yours" — /16 ≈ 65,536 addresses, /24 = 256.

- VPC CIDR can be **/16 (largest) to /28 (smallest)**.
- **AWS reserves 5 IP addresses in every subnet** (network, router, DNS, future, broadcast) — a /24 gives you 251 usable, not 256.
- **No-overlap rule**: VPCs you want to connect (peering, TGW, VPN to on-prem) must have **non-overlapping CIDRs**. Plan ranges up front; you can't peer 10.0.0.0/16 with 10.0.0.0/16.

## The two guards: Security Groups vs NACLs

Two firewalls, different posts, different memories:

| | Security Group (SG) | Network ACL (NACL) |
|---|---|---|
| Guards | The **instance** (its network interface) | The **subnet** boundary |
| State | **STATEFUL** — remembers | **STATELESS** — no memory |
| Rules | **Allow only** (default deny) | Allow **and DENY** |
| Order | All rules evaluated together | **Numbered**, lowest first, first match wins |
| Can reference | **Other SGs** | Only CIDR blocks |

- **Stateful** = a receptionist **with memory**: if she let your request out, she automatically lets the reply back in. With SGs you never write a rule for return traffic.
- **Stateless** = a gate guard with **no memory**: he checked the outgoing request, but the reply is a brand-new stranger to him — it needs its **own explicit rule**. Replies come back on **ephemeral ports (1024–65535)**, so NACLs need an outbound (and inbound, for the reverse direction) allow for that range. THE trap: *"requests go out but responses never return"* → **NACL is missing the ephemeral-port outbound rule**.
- **SG referencing**: instead of IPs, an SG rule can say "allow port 3306 **from the app-tier's security group**." *"Only app servers may reach the database"* → **DB SG allows inbound from the app SG** — elegant, survives IP changes.
- SGs **cannot deny**. THE trap: *"block traffic from a specific malicious IP"* → **NACL DENY rule** (a SG physically has no deny button).

## VPC Endpoints — private hallways to AWS services

Normally, a private instance calling S3 would go out through NAT → internet → S3. Wasteful and exposed. VPC Endpoints let you reach AWS services **without leaving the AWS network**:

| | Gateway Endpoint | Interface Endpoint (PrivateLink) |
|---|---|---|
| Services | **S3 and DynamoDB ONLY** | Nearly **any** AWS service (+ SaaS) |
| Cost | **FREE** | Paid (per hour + per GB) |
| How | A **route-table entry** | An **ENI** (private network card) in your subnet |
| Reach | **VPC-local only** — NOT usable from peered VPCs or on-prem | **IS reachable** over peering / Transit Gateway / VPN |

- *"Private EC2 needs to reach S3, cheapest / most secure way"* → **Gateway Endpoint**, NOT a NAT Gateway. (NAT charges per GB; Gateway Endpoints are free.) This question is practically guaranteed.
- Because Interface Endpoints work across peering/TGW/VPN, the **centralized endpoints pattern** puts them in one shared VPC for many VPCs to use. And yes — an **S3 *interface* endpoint exists** specifically for the *"on-premises servers need private S3 access"* case, since the Gateway one can't be reached from on-prem.

## Connecting VPCs: three very different tools

### VPC Peering — a 1:1 cable

A direct private link between exactly two VPCs (cross-account and cross-region work fine).

- **NOT transitive**: A↔B and B↔C does NOT give A↔C. Every pair needs its own peering + route-table entries.
- **No CIDR overlap allowed.**
- Cheap (free to create; you pay data transfer). Great for **2–3 VPCs**.
- The mesh math kills it: full mesh of n VPCs = **n(n−1)/2** connections. **10 VPCs = 45 peerings.** 25 VPCs = 300. Nobody manages that → …

### Transit Gateway (TGW) — the hub

A regional **hub-and-spoke router**: attach VPCs, **Site-to-Site VPNs, and Direct Connect**, and everything can talk **transitively** through the hub — one attachment per VPC instead of a mesh.

- Share it **cross-account with AWS RAM** (Resource Access Manager).
- **Peer TGWs across regions** for a global network.
- The **only AWS service supporting multicast**.
- **Per-attachment route tables** let you segment: e.g., prod VPCs can't reach dev VPCs even though both hang off the hub.
- Keyword: *"many VPCs (and/or on-prem) need to communicate, minimal management"* → **Transit Gateway**. "Many" starts around 4+; at 25 it's not even a question.

### PrivateLink — a service window, not a network marriage

Peering/TGW **merge networks** — everything can reach everything. PrivateLink instead opens **one service window**: the provider puts an **NLB** in front of their app and creates an **endpoint service**; consumers create an **interface endpoint** in their own VPC and reach *that one service, one-direction, and nothing else*.

- **Overlapping CIDRs are fine** (no routing between networks — it's just an ENI).
- Scales to **thousands of consumer VPCs** without route-table sprawl.
- Keywords: *"SaaS provider exposes a service to many customer VPCs"*, *"partner must access ONE API only, not our whole network"* → **PrivateLink**.

Quick chooser: **2–3 VPCs, full network access → Peering. Many VPCs + VPN/DX, transitive → TGW. Expose one service, no network merge, CIDRs overlap → PrivateLink.**

## Observability & admin access

- **VPC Flow Logs** capture traffic **metadata** — source/destination IP, port, protocol, **ACCEPT or REJECT** — to CloudWatch Logs or S3. Not packet contents. Use for *"why is this connection failing?"* (look for REJECTs → an SG/NACL is blocking) and security analysis. Need the **full packets**? That's **Traffic Mirroring** (copies packets to an inspection appliance).
- **Bastion host** = a DIY **EC2 jump box in a public subnet**: you SSH to it, then hop to private instances. You build, patch, and guard it. The SG chain: bastion's SG allows port 22 **from your corporate IP only**; private instances' SG allows 22 **from the bastion's SG**.
- **SSM Session Manager** = the modern answer: shell access through the Systems Manager agent — **no open port 22, no public IP, no bastion, every session logged** (CloudTrail/S3). THE tie-breaker: *"MOST secure way to administer private instances"* → **Session Manager**, not a bastion.
- **VPC Sharing (via RAM)**: one account owns the VPC, other accounts launch resources into its subnets — **many accounts, one network**, no peering needed. *"Central network team, application accounts deploy into shared subnets"* → VPC sharing.

# Hybrid Networking & Connectivity — SAA

## 1. Core idea

Hybrid connectivity means connecting an **on-premises network** to AWS.

Main services/concepts:

- **AWS Direct Connect (DX)** — dedicated private network connection from on-premises to AWS.
- **Site-to-Site VPN** — encrypted connection over the public Internet.
- **VPC Peering** — private connectivity between two VPCs.
- **VPN CloudHub** — connects multiple remote networks/sites using VPN gateways.
- **Fault tolerance** — use independent connections so one failure does not isolate the VPC.

## 2. Direct Connect and Virtual Interfaces (VIF)

Direct Connect has two layers:

```text
On-premises
    |
    | Physical Direct Connect connection
    v
 AWS Direct Connect
    |
    | Private VIF (logical interface)
    v
   VPC
```

- **Direct Connect connection (DX)** = the physical/dedicated network connection between on-premises and AWS.
- **Virtual Interface (VIF)** = a logical interface configured on top of the Direct Connect connection. It uses **BGP** to exchange routes.
- **Private VIF** = used to access VPC resources using private connectivity.

Mental model:

> **Direct Connect = physical cable; VIF = logical lane on that cable.**

A VIF alone does not remove a physical single point of failure:

```text
DX #1
  |
 VIF #1
 VIF #2
  |
 VPC
```

For fault tolerance, use independent Direct Connect connections with VIFs:

```text
                DX #1
On-premises ---------------\
                            > VPC-1
On-premises ---------------//
                DX #2
```

Each Direct Connect connection can have its own VIF(s).

## 2. Direct Connect redundancy

A single Direct Connect connection is a **single point of failure**.

Example:

```text
On-premises
     |
     | DX #1
     v
   VPC-1
```

If DX #1 fails, on-premises cannot reach VPC-1 through that path.

A redundant design can use another independent connection:

```text
              DX #1
On-premises -----------\\
                        > VPC-1
On-premises -----------//
              DX #2
```

For an SAA question asking how to increase fault tolerance to **VPC-1**, another Direct Connect connection/private virtual interface connected to **VPC-1** is a strong answer.

## 2.1 Why the Private VIF targets the VPC's Region

A **private VIF connects to a Virtual Private Gateway (VGW)**, and the VGW belongs to a VPC in a specific AWS Region.

Mental model:

```text
On-premises
     |
     | Direct Connect
     v
Private VIF
     |
     v
VGW in the VPC's Region
     |
     v
VPC-1
```

If **VPC-1 is in `us-east-1`**, the private VIF used for direct connectivity to VPC-1 must target the **VGW associated with VPC-1 in `us-east-1`**.

```text
On-premises
     |
     | DX
     v
Private VIF
     |
     v
VGW (us-east-1)
     |
     v
VPC-1 (us-east-1)
```

Why this matters in SAA questions:

- A **VPC is regional**.
- A **VGW belongs to a specific VPC/Region**.
- A **private VIF is configured to provide connectivity to that VGW**.
- Therefore, when the answer says **"establish another Direct Connect connection and private virtual interface in the same AWS Region as VPC-1,"** it means the new private VIF must provide connectivity to **VPC-1's VGW**.
- Connecting the new DX/VIF only to **VPC-2** does not create a redundant path to VPC-1 because **VPC Peering is not transitive**.

> **Exam memory:** For redundancy to a specific VPC, the new DX + private VIF must actually reach that VPC's VGW in its Region.

**Important nuance:** Direct Connect is not simply a physical cable that must be geographically located in the same AWS Region. The key point is that the **private VIF's AWS-side target (such as the VGW) must serve the target VPC in its Region**.

## 3. VPN as a backup

You can also use a Site-to-Site VPN as an independent backup path:

```text
                Direct Connect
On-premises --------------------> VPC-1
     \
      \\ Internet VPN
       -------------------------> VPC-1
```

If Direct Connect fails, the VPN can still provide connectivity.

### Mental model

**DX + VPN = two different connectivity paths.**

## 4. VPC Peering

VPC peering provides private connectivity between two VPCs.

```text
VPC-1 <--------> VPC-2
```

However, **VPC peering is not transitive**.

That means you cannot use VPC-2 as a transit router to reach VPC-1 from another network.

For example:

```text
On-premises
     |
     | VPN / DX
     v
   VPC-2
     |
     | VPC Peering
     v
   VPC-1
```

You should **not** assume this gives:

```text
On-premises -> VPC-2 -> VPC-1
```

as a transit path.

### Exam memory

> **VPC Peering is not transitive.**

## 5. Exam question pattern

### Scenario

```text
VPC-1: private subnets
VPC-2: public subnets

VPC-1 <---- VPC Peering ----> VPC-2

On-premises ---- Direct Connect ----> VPC-1
```

Question:

> Which options increase the fault tolerance of the connection to VPC-1?

### Correct choices

**1. Site-to-Site VPN directly between VPC-1 and on-premises**

```text
On-premises
   |\\
   | \\ VPN
   |  \\----> VPC-1
   |
   \-------> DX ----> VPC-1
```

**2. Another Direct Connect connection/private virtual interface to VPC-1**

```text
On-premises
    |\\
 DX#1| \\ DX#2
    |  \\
    +---> VPC-1
```

### Incorrect reasoning

A new VPN or Direct Connect connection to **VPC-2** does not automatically provide transit connectivity to VPC-1 through the peering connection.

```text
On-premises -> VPC-2 -> VPC-1
                    X
             Not transitive
```

## 6. Direct Connect vs VPN

| Feature | Direct Connect | Site-to-Site VPN |
|---|---|---|
| Connection | Dedicated network connection | Internet-based |
| Encryption | Not inherently encrypted | Encrypted with IPsec |
| Reliability | Can be highly reliable with redundancy | Depends partly on Internet path |
| Main use | Consistent private hybrid connectivity | Secure connectivity / backup path |
| Redundancy | Use multiple DX connections/locations | Use redundant VPN tunnels/connections |

## 7. VPC Peering vs Transit Gateway

| | VPC Peering | Transit Gateway |
|---|---|---|
| VPC-to-VPC connectivity | Yes | Yes |
| Transitive routing | **No** | **Yes** |
| Scale | Many point-to-point connections can become complex | Centralized hub model |
| Typical mental model | Direct connection | Network hub |

## 8. Exam memory

- **Single DX = possible single point of failure.**
- **DX + VPN = redundant paths.**
- **Another DX to the same target VPC = redundancy.**
- **VPC Peering is not transitive.**
- A connection to **VPC-2** does not become a connection to **VPC-1** merely because VPC-1 and VPC-2 are peered.
- When a question asks for **fault tolerance to a specific VPC**, look for an additional independent path that actually reaches that VPC.

## 9. One-line mental model

```text
Hybrid connectivity = On-premises <-> AWS

DX   = dedicated private path
VPN  = encrypted Internet path
Peering = direct VPC-to-VPC path, NOT a transit path
``` 

## Question patterns

> *"Block all traffic from a specific IP address"* → **NACL deny rule** (security groups can't deny)

> *"Only the app tier may connect to the database"* → **DB security group allows inbound from the app tier's SG** (SG-to-SG reference)

> *"Private EC2 must access S3 privately at the lowest cost"* → **Gateway VPC Endpoint** (free — NAT Gateway is the expensive decoy)

> *"25 VPCs plus on-premises via VPN must all communicate"* → **Transit Gateway** (peering mesh = 300 connections; TGW is the hub)

> *"Expose one application to a partner VPC; CIDR ranges overlap"* → **PrivateLink** (NLB + endpoint service; overlap doesn't matter)

> *"Single NAT Gateway — make the design highly available"* → **One NAT Gateway per AZ**, each private subnet routing to its local one

> *"Private instances need to download OS updates"* → **NAT Gateway in the public subnet** + private route `0.0.0.0/0 → NAT`

> *"Most secure way for admins to access private instances"* → **SSM Session Manager** (no port 22, no bastion, fully logged)

> *"Determine why traffic to an instance is being rejected"* → **VPC Flow Logs** (metadata shows ACCEPT/REJECT; full packets = Traffic Mirroring)

> *"IPv6-only instances need outbound internet, no inbound"* → **Egress-only Internet Gateway** (IPv6's "NAT")

> *"NAT instance stopped forwarding traffic"* → **Disable the source/destination check**

> *"Outbound requests succeed but responses never arrive"* → **NACL missing outbound ephemeral-port (1024–65535) rule** (stateless!)

## Pocket card

| Keyword | Answer |
|---|---|
| Public subnet definition | Route to IGW |
| Outbound-only internet for private subnet | NAT Gateway (in a public subnet) |
| NAT high availability | One NAT Gateway per AZ |
| NAT instance fix | Disable source/dest check |
| IPv6 outbound-only | Egress-only IGW |
| Reserved IPs per subnet | 5 |
| VPC CIDR size | /16 to /28 |
| Block a specific IP | NACL (deny) |
| Stateful / instance-level | Security Group |
| Stateless / subnet-level / ephemeral ports | NACL |
| Tier-to-tier access control | SG referencing another SG |
| S3/DynamoDB private + free | Gateway Endpoint |
| Endpoint reachable from on-prem/peered VPC | Interface Endpoint (PrivateLink) |
| 2–3 VPCs, full access | VPC Peering (not transitive!) |
| Many VPCs + VPN/DX, transitive hub | Transit Gateway (RAM to share, only multicast) |
| One service, overlapping CIDRs, SaaS | PrivateLink (NLB + endpoint service) |
| Traffic metadata, accept/reject | Flow Logs |
| Full packet capture | Traffic Mirroring |
| Most secure admin access | SSM Session Manager |
| One network, many accounts | VPC Sharing via RAM |

You now own the network — next up are the roads leading into it from the outside world: Route 53, Direct Connect, and Site-to-Site VPN, plus the load balancers from Section 5 that stand at its public doors.
