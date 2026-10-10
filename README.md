# AWS Solutions Architect Associate (SAA-C03) — Topic Guides

**38 exam-focused study guides covering the major AWS Solutions Architect Associate (SAA-C03) topics** — explained the way a good teacher explains them, rather than reading like service documentation.

> Community study notes shared to help others prepare for and pass the exam. 🎉

## What makes these different

Every guide follows the same teaching format:

1. **The idea** — each service explained from zero using a real-world analogy (Multi-AZ = a spare tire, Read Replicas = extra checkout lanes, Route 53 = a phonebook, NAT = a receptionist handling outgoing requests).
2. **Core concepts** — exam-relevant facts, limits, defaults, distinctions, and explicit **"THE trap"** callouts highlighting common mistakes.
3. **Question patterns** — realistic exam-style scenarios, with answers and the keywords that help identify the correct solution.
4. **Pocket card** — a keyword → answer table for rapid revision.

The guides emphasize decision patterns that commonly matter in SAA-C03 scenarios, such as:

- Multi-AZ vs. Read Replicas
- Gateway vs. Interface VPC Endpoints
- SQS vs. SNS vs. Kinesis vs. EventBridge
- RDS vs. Aurora vs. DynamoDB
- NAT Gateway vs. VPC Endpoints
- CloudFront vs. Global Accelerator
- Backup and restore vs. pilot light vs. warm standby

The goal is to understand **why a solution fits the requirements**, not simply memorize service definitions.

## Start here

📖 **[Topic Guides Index](topic-guides/00-README.md)** — all 38 guides, with a suggested reading order organized around exam objectives and study priorities.

### High-value guides to prioritize

If you're short on time, consider starting with these topics. This is a practical study recommendation, not an official ranking of individual service question frequency.

- [VPC](topic-guides/06-VPC.md) — networking fundamentals, routing, endpoints, connectivity, and security controls.
- [S3](topic-guides/02-S3.md) — storage classes, encryption, lifecycle policies, replication, and data protection.
- [RDS & Aurora](topic-guides/09-RDS-Aurora.md) — Multi-AZ, Read Replicas, failover, scaling, and database selection.
- [SQS, SNS & Kinesis](topic-guides/17-SQS-SNS-Kinesis.md) — messaging, event-driven architectures, streaming, and decoupling.
- [Exam Traps & Key Patterns](topic-guides/36-Exam-Traps-KeyPatterns.md) — review after studying the underlying services, then revisit during final revision.

### Official exam-domain weights

AWS organizes the scored SAA-C03 content into four domains:

| Exam domain | Weight |
|---|---:|
| Design Secure Architectures | 30% |
| Design Resilient Architectures | 26% |
| Design High-Performing Architectures | 24% |
| Design Cost-Optimized Architectures | 20% |

These percentages describe the official exam domains, not the frequency of individual services or question patterns. Use the weights to guide your overall preparation, but don't neglect a domain or service solely because it seems less prominent.

**Official reference:** [AWS Certified Solutions Architect – Associate (SAA-C03) Exam Guide](https://docs.aws.amazon.com/aws-certification/latest/solutions-architect-associate-03.html)

## Exam-technique rules

### 1. Identify the actual requirement

You can start by reading the final sentence to identify what the question asks for. Then read the entire scenario carefully.

Distinguish between requirements such as:

- **MOST cost-effective**
- **LEAST operational overhead**
- **HIGHLY available**
- **MINIMUM latency**
- **NEAR-ZERO data loss**
- **MINIMUM recovery time**

The best solution depends on the complete set of requirements, not just the service named in the question.

### 2. Count every requirement

Treat phrases such as "X as well as Y" as a checklist. The correct answer must satisfy every required condition.

If a question asks for both private connectivity and high availability, an option that provides only private connectivity is incomplete.

### 3. Pay attention to qualifying words

Words and phrases such as **serverless, automatically, at no additional cost, immediately, directly, cannot, and always** can change whether an answer is correct.

Don't reject an answer simply because it uses an absolute term. Check whether the statement is actually true for the specified service, configuration, and situation.

### 4. Compare similar answer choices carefully

When two options appear almost identical, examine their differences:

- Which one satisfies all requirements?
- Which one introduces an unnecessary component?
- Which one requires more operational work?
- Which one fails a stated performance, availability, security, or cost constraint?

A single qualification can determine the correct answer.

### 5. Use numerical requirements to eliminate options

Pay attention to stated limits and targets, including:

- Retrieval time and storage durability
- IOPS and throughput
- Recovery Point Objective (RPO) and Recovery Time Objective (RTO)
- Timeout limits
- Data-transfer volume
- Capacity and scaling requirements

Use the values to eliminate incompatible solutions before comparing the remaining options.

Verify unfamiliar or potentially outdated numbers against current AWS documentation.

### 6. Don't guess based on how familiar an option sounds

An unfamiliar service feature can be correct, but unfamiliarity alone doesn't make an option more likely to be right.

If none of the familiar choices appears suitable:

1. Re-read the exact requirement.
2. Identify which requirement eliminates each plausible option.
3. Check whether an unfamiliar service or feature fits the scenario.
4. Verify its actual behavior and limitations if necessary.

Choose the answer that satisfies the requirements, not the one that sounds most sophisticated or most familiar.

### 7. Prefer the simplest solution that meets every requirement

When several options work, consider operational overhead, cost, availability, scalability, and security.

Don't automatically select the cheapest service if it fails a reliability or performance requirement. Likewise, don't select a more complex architecture when a managed service meets all the stated requirements.

## Accuracy and disclaimer

These are community study notes, not official AWS exam content. They are not affiliated with or endorsed by Amazon Web Services.

The guides are intended to explain concepts and help you practice architectural decisions. They should not be treated as a guarantee of complete exam coverage or as a substitute for the official exam guide.

AWS service features, quotas, pricing, regional availability, and product lifecycles can change. Before relying on a specific limit or behavior:

- Check the current official AWS documentation.
- Verify that the service and feature remain supported.
- Distinguish default settings from maximum configurable limits.
- Pay attention to exceptions and prerequisites.
- Use the current SAA-C03 exam guide to confirm the scope of your preparation.

**Official exam guide:** [AWS Certified Solutions Architect – Associate (SAA-C03)](https://docs.aws.amazon.com/aws-certification/latest/solutions-architect-associate-03.html)

## License and attribution

This repository is distributed under the [Creative Commons Attribution 4.0 International (CC BY 4.0)](LICENSE) license.

You may share and adapt these guides, including for commercial purposes, provided you follow the license terms. When redistributing or building upon the material, give appropriate credit to **RonitSachdev**, link to the [original repository](https://github.com/RonitSachdev/aws-saa-c03-guides) and the [CC BY 4.0 license](https://creativecommons.org/licenses/by/4.0/), and indicate whether you made changes.

If these guides help with your preparation, consider giving the original repository a ⭐.

