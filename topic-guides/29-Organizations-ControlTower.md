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

## Question patterns

> *"Prevent all accounts in the organization from launching resources outside approved regions"* → **SCP on the root/OU** (org-wide ceiling = SCP)

> *"An SCP denies an action, yet the management account can still perform it — why?"* → **SCPs don't apply to the management account** (the exemption trap)

> *"New teams need AWS accounts that come pre-configured with security baselines, via self-service"* → **Control Tower Account Factory** (standardized account vending)

> *"Company wants a single bill and to share Reserved Instance discounts across accounts"* → **Organizations consolidated billing** (pooled usage, shared RI/SP)

> *"Multiple accounts must use the same VPC subnets / a central Transit Gateway"* → **AWS RAM** (share, don't duplicate)

> *"IAM user has full admin policy in a member account but is denied — what to check?"* → **The SCP** (ceiling overrides IAM allow)

> *"Automatically set up a multi-account environment following AWS best practices"* → **Control Tower** (landing zone + guardrails)

> *"Guardrail that flags noncompliant resources but doesn't block creation"* → **Detective guardrail** (Config rule; preventive = SCP = block)

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

Governance says which accounts may do what — the next layer down is letting teams safely launch only pre-approved infrastructure, which is Service Catalog's job.
