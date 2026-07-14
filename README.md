# AWS Solutions Architect Associate (SAA-C03) — Complete Topic Guides

**38 exam-focused study guides covering the entire SAA-C03 syllabus** — written the way a good teacher explains, not the way documentation reads.

> Battle-tested on a successful exam attempt — shared in the hope they help you pass too. 🎉

## What makes these different

Every guide follows the same battle-tested format:

1. **The idea** — each service explained *from zero* with a real-world analogy (Multi-AZ = a spare tire, Read Replicas = extra checkout lanes, Route 53 = a phonebook, NAT = a receptionist mailing your letters…)
2. **Core concepts** — the exam-tested facts and numbers, with explicit **"THE trap"** callouts for the mistakes the exam is designed to harvest
3. **Question patterns** — realistic exam-style scenarios with the answer *and the signal keywords that crack them*
4. **Pocket card** — a keyword → answer table for rapid review

The guides teach the *decision patterns* the exam actually tests ("Multi-AZ vs Read Replica", "Gateway vs Interface Endpoint", "SQS vs SNS vs Kinesis vs EventBridge") — not service trivia in isolation.

## Start here

📖 **[Topic Guides Index](topic-guides/00-README.md)** — all 38 guides with a suggested reading order grouped by exam weight.

The heavy hitters, if you're short on time:
- [VPC](topic-guides/06-VPC.md) — the single biggest exam topic
- [S3](topic-guides/02-S3.md) — storage classes, encryption, the works
- [RDS & Aurora](topic-guides/09-RDS-Aurora.md) — home of the most-tested distinction on the exam
- [SQS, SNS & Kinesis](topic-guides/17-SQS-SNS-Kinesis.md) — the messaging block
- [Exam Traps & Key Patterns](topic-guides/36-Exam-Traps-KeyPatterns.md) — ⭐ read this last, and again on exam morning

## Exam-technique rules

1. **Read the last sentence of the question first** — "MOST cost-effective" vs "LEAST operational overhead" vs "highly available" decides between the two plausible finalists.
2. **Count the requirements** — "X *as well as* Y" is a checklist; the right answer ticks every box. Half-solutions are planted for people who stop at the first match.
3. **Scan options for poison qualifiers** — *serverless, automatically, at no cost, immediately, directly, cannot/always* — a true fact welded to one false word is the exam's favorite distractor.
4. **When two options are twins**, the entire question lives in the differing clause.
5. **Requirements are eliminators, not decoration** — a stated number (retrieval time, IOPS, RPO/RTO) exists to kill specific options. Eliminate first, then pick the cheapest/most-managed survivor.
6. **When every familiar option is wrong-scope, the unfamiliar one is the answer** — real-but-unknown features sound specific and boring; fabricated ones sound grand and vague.

## Disclaimer

Community study notes — not affiliated with or endorsed by AWS. Accurate to the SAA-C03 exam as of mid-2026; always cross-check details that matter against current AWS documentation.

## License

[CC BY 4.0](LICENSE) — you're free to use, share, and adapt these guides, **but you must give credit**: mention **RonitSachdev** and link back to this repository in anything you build from them. If these helped you pass, a ⭐ on the repo is appreciated too.
