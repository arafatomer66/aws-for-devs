# AWS Pricing Models

**TL;DR** — On-Demand = pay per second. Reserved / Savings Plans = commit for 1–3 years, save 30–72%. Spot = use spare capacity, save up to 90%, but can be reclaimed.

## The four buying models for compute

### 1. On-Demand
Pay-as-you-go. Per-second billing for EC2 (min 60s), per-ms for Lambda.

- **Pros:** No commitment, full flexibility.
- **Cons:** Most expensive.
- **Use for:** Unpredictable workloads, short-lived dev/test, anything you might delete next week.

### 2. Reserved Instances (RIs)
Commit to a specific instance type/family for 1 or 3 years in advance. Save 30–60%.

- **Standard RI:** Locked to instance family. Biggest savings.
- **Convertible RI:** Can swap family later. Smaller savings.
- **Payment:** All Upfront (max savings) / Partial / No Upfront.
- Mostly **superseded by Savings Plans** for compute — RIs still matter for RDS, ElastiCache, OpenSearch, Redshift.

### 3. Savings Plans
Commit to **a dollar amount per hour** (e.g., $10/hr) for 1 or 3 years. AWS applies the discount to whatever compute you use up to that amount.

- **Compute Savings Plan:** Most flexible — covers EC2 + Fargate + Lambda, any region, any family. Save up to 66%.
- **EC2 Instance Savings Plan:** Locked to a family in a region. Save up to 72%.
- **SageMaker Savings Plan:** Just for SageMaker.

**Rule of thumb:** If you have steady baseline load, Savings Plans pay back in 6–9 months.

### 4. Spot Instances
Bid for **unused EC2 capacity** at up to **90% off**. AWS can reclaim with 2 minutes' notice.

- **Use for:** Stateless workers, batch jobs, CI/CD runners, big-data processing, ML training with checkpointing, Fargate Spot for ECS.
- **Don't use for:** Stateful databases, anything where reclamation hurts users.
- Handle interruption gracefully — listen for the spot termination notice on instance metadata.

## Cost comparison example

A `m6i.large` (Linux, us-east-1):

| Model | Hourly | Monthly | Yearly | Savings |
|---|---|---|---|---|
| On-Demand | $0.096 | $69 | $841 | — |
| 1-yr Savings Plan, No Upfront | $0.060 | $43 | $525 | 38% |
| 3-yr Savings Plan, All Upfront | $0.027 | $19 | $235 | 72% |
| Spot | ~$0.029 | ~$21 | ~$254 | ~70% (variable) |

## Free Tier (12-month + always-free)

See [accounts-billing.md](./04-accounts-billing.md#the-free-tier).

## Other pricing levers

- **Graviton (ARM)** — same workload, ~20% cheaper than x86. Just flip the architecture.
- **Storage tiers** — S3 Standard → Standard-IA → Glacier Instant → Glacier Deep Archive (90% cheaper).
- **Auto-scaling** — pay for capacity only when you have demand.
- **Right-sizing** — Compute Optimizer recommends smaller instances if you're underutilized.
- **Idle resources** — kill unattached EBS volumes, idle NAT Gateways, unused EIPs.

## Free vs. always-billed services

**Always free** — no charge for use, only for what you do with them:
- IAM, Organizations, AWS CLI, SDKs.
- VPC itself (you pay for what's *inside*: NAT, ELB, etc.).
- CloudFormation, CDK templates (you pay for resources created).

**Sneaky always-paid** to watch:
- NAT Gateway: ~$32/mo idle.
- ELB: ~$16/mo idle.
- Public IPv4 addresses: **$0.005/hr each** (added 2024 — adds up).
- KMS customer-managed keys: $1/key/mo.

## Pricing tools

- **AWS Pricing Calculator** — `calculator.aws` — estimate before you build.
- **Cost Explorer** — analyze actual spend.
- **Budgets** — alert on overspend.
- **Compute Optimizer** — right-sizing recommendations.
- **Trusted Advisor** — finds idle resources.

## Gotchas

- **Egress is the killer.** Inbound is free; outbound to internet is ~$0.09/GB. CloudFront is cheaper egress per GB, sometimes free between AWS services.
- **Data transfer between AZs is NOT free** — $0.01/GB each way.
- **CloudWatch Logs ingestion costs $0.50/GB.** Verbose logging at scale gets expensive.
- **Lambda is billed per ms × memory** — over-provisioning RAM costs more.
- **You pay for Reserved Instances and Savings Plans whether you use them or not.** Don't over-commit.

## Related

- [Cost Explorer](../11-cost-management/cost-explorer.md)
- [Compute Optimizer](../11-cost-management/compute-optimizer.md)
- [Budgets](../11-cost-management/budgets.md)
