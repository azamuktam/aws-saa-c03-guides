# Section 36: Classic Traps & Must-Know Combos

## The idea

The SAA-C03 isn't trying to see if you've *heard* of services — it's checking whether you know where the **tripwires** are. Think of this section as the **map of the minefield**: every trap below is a wrong answer that *sounds* right and appears on the exam over and over, paired with the truth that defuses it. Walk this list twice before exam day and you'll feel the wires before you step on them.

## The greatest-hits trap list

| THE trap | The truth |
|---|---|
| "Read from the RDS Multi-AZ standby to offload queries" | **NO** — the standby is invisible; use **read replicas** for reads |
| "CNAME record at the zone apex (example.com)" | CNAMEs can't sit at the apex — use an **Alias record** |
| "Add a deny rule to the security group" | Security groups have **no deny rules** (allow-only, stateful) — deny lives in **NACLs** |
| "Private subnet reaches S3 via NAT Gateway" | Works but costs money — **Gateway VPC Endpoint** is free and private |
| "Run the 20-minute job in Lambda" | Lambda caps at **15 minutes** — use Fargate/Batch/Step Functions |
| "Mount EFS on Windows instances" | EFS is Linux-only — Windows shared storage = **FSx for Windows File Server** |
| "Snowball imports directly into Glacier" | No — Snowball lands in **S3**, then a **lifecycle policy** moves it to Glacier |
| "A↔B and B↔C peering lets A reach C" | Peering is **non-transitive** — need **Transit Gateway** (or direct A↔C peering) |
| "Encrypt the existing unencrypted RDS/EBS in place" | Impossible in place — **snapshot → copy snapshot with encryption → restore** |
| "KMS for FIPS 140-2 Level 3 / single-tenant keys" | That's **CloudHSM** (KMS is Level 2*, multi-tenant) |
| "Shield Standard stops layer-7 application attacks" | Standard is L3/L4 only — L7 protection = **WAF / Shield Advanced** |
| "Apply an SCP to restrict the management account" | SCPs **never affect the management account** |
| "Replay DynamoDB Streams data from last month" | Streams keep **24 hours** — longer retention → **Kinesis Data Streams** |
| "Give the ALB a static IP" | ALBs have no static IPs — **NLB** has static/Elastic IPs (or NLB in front of ALB) |
| "Add an LSI to the existing DynamoDB table" | **LSIs only at table creation** — after creation, add a **GSI** |
| "Accelerate UDP traffic with CloudFront" | CloudFront is HTTP(S) only — UDP/TCP acceleration = **Global Accelerator** |
| "Parameter Store rotates the DB password automatically" | No native rotation — automatic rotation = **Secrets Manager** |
| "One NAT Gateway serves all AZs, highly available" | AZ failure kills it — deploy **one NAT Gateway per AZ** |
| "SNS alone guarantees consumers never miss messages" | SNS doesn't persist for offline consumers — durable fan-out = **SNS → SQS** |
| "Direct Connect is encrypted because it's private" | **Not encrypted** — run a **VPN over DX** for encryption |

## Must-know combos (the exam's favorite Lego sets)

| Need | The combo |
|---|---|
| Serverless web app | **CloudFront + S3 (static) + API Gateway + Lambda + DynamoDB** |
| Reliable fan-out | **SNS → multiple SQS queues** (durable, each consumer own queue) |
| Global highly available DB | **Aurora Global Database** (SQL) / **DynamoDB Global Tables** (NoSQL) |
| Protect a web app | **CloudFront + WAF + Shield + ALB** |
| Real-time streaming pipeline | **Kinesis Data Streams → Lambda → DynamoDB/S3** |
| Analytics on data lake | **S3 + Glue (catalog/ETL) + Athena (query) + QuickSight (dashboards)** |
| Migration toolkit | **DataSync (files) + DMS (databases) + DX/Snow family (transport)** |
| DR stack | **Route 53 failover + Aurora Global + S3 CRR** |
| App authentication | **Cognito User Pools (login) + Identity Pools (AWS credentials)** |

## Exam technique (the meta-game)

- **Read the last sentence first** — that's where the actual question and qualifier live; the story above it is scenery.
- **Eliminate two** — almost every question has two obviously wrong answers. Kill them, then compare the survivors.
- **The qualifier decides** between two working answers: **cost** → cheapest that still works; **least operational overhead** → most managed/serverless; **highly available** → Multi-AZ; **survive region failure** → multi-region.
- **~2 minutes per question** (130 min, 65 questions). Stuck at 90 seconds? **Flag it and move on** — later questions often jog your memory.
- **Answer everything.** No penalty for wrong answers; a guess beats a blank.

## Question patterns

> *"Route example.com (apex) to an ALB"* → **Alias record** (CNAME illegal at apex)

> *"Block a specific malicious IP address"* → **NACL deny rule** (security groups can't deny)

> *"Private subnet needs S3 access at no cost"* → **Gateway VPC Endpoint** (NAT Gateway is the paid decoy)

> *"Encrypt an existing unencrypted RDS instance"* → **Snapshot → encrypted copy → restore** ("enable encryption" checkbox doesn't exist)

> *"Fan out orders to inventory, billing, shipping — none may be lost"* → **SNS topic → three SQS queues** (durability per consumer)

> *"Static IPs required by the firewall team for the load balancer"* → **Network Load Balancer** (ALB has none)

> *"Improve performance AND static IPs for global TCP/UDP app"* → **Global Accelerator** (CloudFront can't do UDP)

> *"Rotate database credentials automatically every 30 days"* → **Secrets Manager** (Parameter Store is the decoy)

## Pocket card

| Keyword | Answer |
|---|---|
| Apex domain | Alias record |
| Deny traffic | NACL (never SG) |
| Free S3 access from VPC | Gateway Endpoint |
| >15 minutes | Not Lambda (Fargate/Batch) |
| Windows file share | FSx for Windows |
| Transitive routing | Transit Gateway |
| Encrypt existing volume/DB | Snapshot dance |
| FIPS Level 3 | CloudHSM |
| Static IP load balancer | NLB |
| UDP acceleration | Global Accelerator |
| Auto-rotation | Secrets Manager |
| HA NAT | One per AZ |
| Durable fan-out | SNS → SQS |
| Encrypted DX | VPN over DX |

Traps mapped, combos memorized — one last sweep remains: the small-but-tested gap-fill services that each show up for one or two easy points.
