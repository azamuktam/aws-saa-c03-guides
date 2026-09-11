# Section 24: Directory Services

## The idea

First, what *is* Active Directory? **Active Directory (AD)** is Microsoft's system for centrally managing **users, computers, groups, passwords, and access** inside an organization. A company can have an AD domain such as `corp.example.com`, and Windows computers can join that domain and authenticate users against AD. **LDAP** is a protocol commonly used to communicate with directory services, and **domain join** means connecting a machine to the AD domain.

Now the AWS problem: a company has workloads in AWS, but some services still need **Microsoft AD** for authentication. Examples include **FSx for Windows File Server, Amazon WorkSpaces, RDS for SQL Server, Windows EC2 instances**, and applications that use LDAP or Kerberos.

AWS gives you **three main options**:

- **AWS Managed Microsoft AD** = AWS provides a **real Microsoft Active Directory** in AWS. It supports features such as domain joining, Group Policy, LDAP, and Kerberos. It can also establish a trust relationship with an existing on-premises AD.
- **AD Connector** = connects AWS services to an **existing on-premises Microsoft AD**. It does not create or store a separate directory in AWS; authentication requests are forwarded to the existing AD.
- **Simple AD** = a **basic directory service** based on Samba. It provides simpler directory functionality but does not provide the full feature set of Microsoft AD.

The key distinction:

**Managed Microsoft AD** → full Microsoft AD in AWS  
**AD Connector** → use existing on-premises AD from AWS  
**Simple AD** → basic directory for simpler workloads

### The three options (the table to memorize)

| | **AWS Managed Microsoft AD** | **AD Connector** | **Simple AD** |
|---|---|---|---|
| What it really is | **REAL Microsoft AD** running in AWS | A **PROXY** — just forwards auth requests | **Samba-based** AD-compatible clone |
| Users stored in AWS? | Yes | **No — nothing stored in AWS** | Yes |
| Works with on-prem AD? | Yes — **TRUST relationship** (two-way trust; use users from **both** sides) | Yes — it *only* redirects to on-prem | **NO trust** — standalone only |
| MFA support | **Yes** | Yes (via on-prem RADIUS) | **No MFA** |
| Best for | **Full AD features in the cloud**; EC2 domain join; RDS/FSx integration; hybrid via trust | **"Keep all identities on-prem"**, let AWS services authenticate against them | **Cheap basic LDAP**, small orgs, **< 5,000 users** |

Jargon check: a **trust relationship** means two separate AD forests agree to honor each other's logins — cloud AD trusts on-prem AD and vice versa (**two-way trust**), so users from *either* directory can access resources on *either* side, without syncing passwords into the cloud.

```
 Managed Microsoft AD          AD Connector              Simple AD
 ┌─────────────────┐         ┌──────────────┐         ┌──────────────┐
 │  Full AD in AWS │◀═trust═▶│  proxy only  │────────▶│ standalone   │
 │  (users stored) │  on-prem│ (no users in │  on-prem│ Samba clone  │
 │                 │    AD   │     AWS)     │    AD   │ no trust/MFA │
 └─────────────────┘         └──────────────┘         └──────────────┘
   full copy w/ trust        phone line to HQ           budget clone
```

### How the exam distinguishes them

- **Managed Microsoft AD** is the answer when the scenario needs *actual AD machinery in AWS*: **EC2 Windows domain join**, **MFA**, seamless integration with **RDS for SQL Server / FSx for Windows / WorkSpaces**, or a **trust with on-prem** so both user sets work. Phrase to spot: *"full AD features in the cloud"* or *"trust with on-prem"*.
- **AD Connector** is the answer when compliance or policy says **identities must never be stored in the cloud**. It's a pipe, not a directory — every authentication is redirected to the on-prem domain controllers. (Corollary: if the VPN/Direct Connect link to on-prem dies, authentication dies with it.)
- **Simple AD** is the answer when the words are **"lowest cost"**, **"basic LDAP"**, **"small organization"** — and *nothing* in the scenario mentions trust, MFA, or on-prem integration.

THE trap: *"connect Simple AD to on-prem AD with a trust"* → **can't**. Simple AD supports **no trust and no MFA** — the moment either word appears, Simple AD is eliminated.

THE trap: *"AD Connector as a standalone directory"* → also can't. A phone line with no head office on the other end connects to nothing — **AD Connector requires an existing on-prem AD**.

### The common pairing pattern

Remember this trio, because it fuels most directory questions: **FSx for Windows File Server, Amazon WorkSpaces, and RDS for SQL Server (Windows Authentication) all require a directory.** The question is rarely "do I need a directory?" — it's *which* of the three options fits the constraints given. FSx + "use our existing on-prem identities" → Managed AD with a trust, or AD Connector. FSx + "need MFA and cloud-resident AD" → Managed Microsoft AD.

**Memory hook:** **Managed AD = full copy with trust; Connector = phone line to on-prem; Simple = budget clone.**

## Question patterns

> *"Deploy FSx for Windows File Server; users must authenticate with existing on-prem AD identities"* → **Managed Microsoft AD with a trust, or AD Connector** (both let on-prem identities work; pick whichever the options offer).

> *"Security policy requires that no user identities are stored in the cloud"* → **AD Connector** (proxy only — all auth redirected on-prem, nothing stored in AWS).

> *"Small company needs a low-cost, basic LDAP-compatible directory for one application"* → **Simple AD** (cheap Samba standalone, fine under ~5,000 users).

> *"EC2 Windows instances must join a domain, and MFA is required"* → **AWS Managed Microsoft AD** (real AD: domain join + MFA; Simple AD has neither MFA nor trust).

> *"Establish a two-way trust between AWS and the on-premises domain so users on both sides access resources"* → **Managed Microsoft AD** (the only option supporting trust relationships).

> *"RDS for SQL Server with Windows Authentication for corporate users"* → **Managed Microsoft AD** (RDS integrates with it directly; add a trust for on-prem users).

> *"WorkSpaces desktops authenticating against on-prem AD without replicating the directory"* → **AD Connector** (phone line: auth flows back on-prem).

## Pocket card

| Keyword | Answer |
|---|---|
| Full AD features in AWS | Managed Microsoft AD |
| Trust relationship with on-prem | Managed Microsoft AD (two-way trust) |
| MFA + domain join | Managed Microsoft AD |
| No identities stored in cloud | AD Connector |
| Proxy / redirect auth to on-prem | AD Connector |
| Low cost, basic LDAP, small org | Simple AD |
| No trust, no MFA | Simple AD's limits |
| FSx Windows / WorkSpaces / RDS SQL Server | All need a directory — pick from the three |
| Memory hook | Full copy w/ trust / phone line / budget clone |

Directories solve *who* on-prem users are in the cloud — next we solve where their *files* live, with the hybrid storage bridge crew: Storage Gateway, DataSync, and Transfer Family.
