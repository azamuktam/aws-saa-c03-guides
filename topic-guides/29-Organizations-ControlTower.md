# Section 29: AWS Organizations & Control Tower

## The idea

This section is about managing many AWS accounts centrally.

The key idea:

**AWS Organizations** = create, group, govern, and centrally manage AWS accounts.

**AWS Control Tower** = use Organizations plus automation/guardrails to quickly build and govern a multi-account AWS environment.

## Core concepts

**Consolidated billing** — the whole organization gets **one bill**, paid by the management account. Better yet, usage is **pooled for volume discounts**, and **Reserved Instances and Savings Plans are shared across accounts** by default: if account A bought an RI it isn't using, account B's matching instance gets the discount. "Multiple accounts, want one invoice and shared discounts" → Organizations consolidated billing.

**SCPs (Service Control Policies)** — the exam's favorite. Three facts to burn in:

- SCPs are **permission CEILINGS**, not grants. They define the *maximum* of what identities in an account **can possibly do** — they **never give anyone permission**. A user still needs an IAM policy that allows the action.
- **Effective permission = SCP allows it AND IAM allows it.** Either one denying = denied.
- They attach to the **Root, OUs, or individual accounts** and cascade down the tree. Classic use: explicit-deny guardrails — "prevent every account from using regions outside eu-west-1," "prevent anyone from disabling CloudTrail."

THE trap: **SCPs do NOT apply to the management account.** The building owner is exempt from the building rules. If a question says "the restriction must also bind the management account," an SCP alone can't do it — and that's the point being tested. (Corollary best practice: keep workloads out of the management account.)

A second sneaky pattern: *"A user has AdministratorAccess in a member account but gets Access Denied."* Nothing is broken — **check the SCP** on their account or OU.

**Control Tower** — Organizations gives you the raw tree; **Control Tower is the automated, opinionated setup on top of it**. It builds a **landing zone** (a pre-architected multi-account environment with logging, audit accounts, SSO), and provides:

| Piece | What it does |
|---|---|
| **Account Factory** | Self-service creation of **new, standardized, pre-configured accounts** |
| **Preventive guardrails** | Implemented as **SCPs** — *block* disallowed actions |
| **Detective guardrails** | Implemented as **AWS Config rules** — *detect and flag* violations after the fact |

Signal phrase: "set up / automate a **governed, secure multi-account environment** with best practices" → **Control Tower**. If the question is just about grouping accounts or billing, plain Organizations suffices.

**RAM (Resource Access Manager)** — share actual **resources** across accounts without duplicating them: **VPC subnets** (multiple accounts launching into one shared VPC), **Transit Gateways**, **Route 53 Resolver rules**, License Manager configs. "Avoid building the same networking in every account" → RAM.

---

## Control Tower drift notifications

Control Tower doesn't just *set up* your multi-account environment — it also **watches it for changes** and can **alert you** when the structure drifts from what's expected.

### What is "drift" in Control Tower?

**Drift** = when the **actual state** of your AWS Organization differs from the **expected state** defined by Control Tower.

```text
Expected state (Control Tower baseline)
              │
              │  someone changes it manually
              ▼
Actual state (drifted)
              │
              ▼
   Control Tower detects drift
```

Common causes of drift:

- An **OU or account is moved** out of its expected OU
- An account is **removed** from an OU
- The **OU hierarchy is changed** manually outside Control Tower
- A **guardrail** is removed or modified

### Account drift notifications

Control Tower can send **drift notifications** when it detects these changes.

| Feature | Details |
|---|---|
| **Detection** | Control Tower continuously monitors OU/account structure |
| **Notification delivery** | **Amazon SNS** topic |
| **Subscription** | Stakeholders subscribe to the SNS topic (email, SMS, Lambda, etc.) |
| **Setup overhead** | Built-in — no custom code or rules |

```text
OU hierarchy / account change
              ↓
   Control Tower detects drift
              ↓
   SNS topic publishes notification
              ↓
   Stakeholders subscribed → alerted
```

### Why this matters for the exam

The trigger phrase:

> **"Monitor changes to the OU hierarchy and allow stakeholders to subscribe to related alerts, with least administrative overhead."**

→ **Control Tower + account drift notifications**

Because:

- Control Tower **automatically monitors** the OU structure
- Notifications flow through **SNS**, which stakeholders can **subscribe** to
- It's a **managed, built-in** feature → **least overhead**

### Drift type comparison (important distinction)

| Type of drift | What it monitors | Tool |
|---|---|---|
| **Control Tower account drift** | **OU hierarchy / account structure** changes | **Control Tower** ✅ |
| **AWS Config drift** | Resource configuration compliance | AWS Config rules |
| **CloudFormation StackSet drift** | Stack resources differ from template | CloudFormation |

Only **Control Tower drift notifications** monitor **OU hierarchy changes** and provide **subscribable alerts** with minimal setup.

### Exam traps

| Distractor | Why it's wrong |
|---|---|
| **Control Tower + AWS Config aggregated rules** | Config rules evaluate **resource compliance**, not **OU hierarchy changes** |
| **Service Catalog + CloudTrail org trail** | Service Catalog creates resources, not accounts; CloudTrail logs but doesn't monitor OU structure or send subscribable alerts |
| **CloudFormation StackSets + drift detection** | StackSet drift is about **stack resources**, not OU hierarchy |

---

## Question patterns

> *"Prevent all accounts in the organization from launching resources outside approved regions"* → **SCP on the root/OU** (org-wide ceiling = SCP)

> *"An SCP denies an action, yet the management account can still perform it — why?"* → **SCPs don't apply to the management account** (the exemption trap)

> *"New teams need AWS accounts that come pre-configured with security baselines, via self-service"* → **Control Tower Account Factory** (standardized account vending)

> *"Company wants a single bill and to share Reserved Instance discounts across accounts"* → **Organizations consolidated billing** (pooled usage, shared RI/SP)

> *"Multiple accounts must use the same VPC subnets / a central Transit Gateway"* → **AWS RAM** (share, don't duplicate)

> *"IAM user has full admin policy in a member account but is denied — what to check?"* → **The SCP** (ceiling overrides IAM allow)

> *"Automatically set up a multi-account environment following AWS best practices"* → **Control Tower** (landing zone + guardrails)

> *"Guardrail that flags noncompliant resources but doesn't block creation"* → **Detective guardrail** (Config rule; preventive = SCP = block)

> *"Monitor changes to the OU hierarchy and let stakeholders subscribe to alerts, with least overhead"* → **Control Tower account drift notifications**

> *"Detect when an account is moved out of its expected OU"* → **Control Tower drift detection**

---

## Pocket card

| Keyword | Answer |
|---|---|
| Block actions org-wide / by OU | SCP |
| SCP grants permissions? | Never — ceiling only (needs IAM allow too) |
| SCP vs management account | Doesn't apply — exempt |
| One bill, shared RI/SP discounts | Consolidated billing |
| Automated governed multi-account setup | Control Tower |
| Self-service standardized new accounts | Account Factory |
| Preventive guardrail | SCP (blocks) |
| Detective guardrail | Config rule (flags) |
| Share subnets / TGW / resolver rules | RAM |
| Admin denied in member account | Check the SCP |
| Monitor OU hierarchy changes + alerts | Control Tower drift notifications |
| Delivers Control Tower drift alerts | Amazon SNS |
| Detects account moved out of OU | Control Tower drift |
| OU structure drift vs resource drift | Control Tower vs AWS Config |

Governance says which accounts may do what — the next layer down is letting teams safely launch only pre-approved infrastructure, which is Service Catalog's job.
