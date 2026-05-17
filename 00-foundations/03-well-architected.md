# Well-Architected Framework

**TL;DR** — AWS's official "how to build things right on AWS" guide. Six pillars. Use it as a checklist before shipping anything serious to production.

## The 6 pillars

### 1. Operational Excellence
Run and monitor systems to deliver business value, continuously improve.

- Automate ops as code (CloudFormation, CDK, Terraform).
- Make frequent, small, reversible changes (CI/CD).
- Refine procedures based on failures (post-mortems).
- Anticipate failure (game days, chaos engineering).

**Key services:** CloudFormation, CDK, CodePipeline, CloudWatch, Systems Manager.

### 2. Security
Protect data, systems, and assets.

- Strong identity foundation (IAM, MFA, least privilege).
- Apply security at all layers (network, host, app, data).
- Automate security best practices.
- Protect data in transit (TLS) and at rest (KMS).
- Prepare for security events (incident response runbooks).

**Key services:** IAM, KMS, Secrets Manager, GuardDuty, Security Hub, WAF.

### 3. Reliability
Workload performs correctly, recovers from failures, scales to demand.

- Test recovery procedures.
- Auto-recover from failure (Auto Scaling, health checks).
- Scale horizontally — many small things over one big thing.
- Stop guessing capacity — use auto-scaling.
- Manage changes through automation.

**Key services:** Auto Scaling, Route 53 health checks, Multi-AZ RDS, S3 cross-region replication.

### 4. Performance Efficiency
Use compute resources efficiently and maintain that as demand changes.

- Democratize advanced tech (use managed services, don't roll your own DB).
- Go global in minutes (CloudFront, Global Accelerator).
- Use serverless where it fits.
- Experiment more often (A/B test infra choices).
- Mechanical sympathy — pick the right tool (e.g. DynamoDB for low-latency K/V, not RDS).

**Key services:** Lambda, CloudFront, DynamoDB, ElastiCache, Compute Optimizer.

### 5. Cost Optimization
Avoid unnecessary cost.

- Adopt a consumption model (pay for what you use).
- Measure overall efficiency.
- Stop spending money on undifferentiated heavy lifting (managed services).
- Analyze and attribute expenditure (tags, Cost Explorer).
- Use the right pricing model (Spot for stateless, Reserved for steady).

**Key services:** Cost Explorer, Budgets, Compute Optimizer, Savings Plans, Trusted Advisor.

### 6. Sustainability
Minimize environmental impact (added 2021).

- Choose efficient regions (some run on more renewables).
- Maximize utilization (right-sizing).
- Use managed services (AWS amortizes the carbon).
- Use Graviton (ARM) — better perf-per-watt than x86.

**Key services:** Customer Carbon Footprint Tool, Graviton, Compute Optimizer.

## How to actually use this

The Well-Architected Tool is a free service in the AWS Console. You answer ~50 questions per pillar, and it gives you a remediation list.

```
AWS Console → Well-Architected Tool → Define workload → Run review
```

Or just ask each question yourself for each pillar:

- **Operational:** "How do we get notified if it breaks at 3 AM?"
- **Security:** "Who has access to production data? Why?"
- **Reliability:** "If an AZ disappears, what happens?"
- **Performance:** "What's our p99 latency? Can we measure it?"
- **Cost:** "What's our cost-per-customer? Per request?"
- **Sustainability:** "Are we using Graviton where we can?"

## Real-world example

> Designing ShareDeal's order service:
>
> - **Operational:** CDK for infra, CodePipeline for deploys, CloudWatch alarms on 5xx rate.
> - **Security:** IAM role per microservice, secrets in Secrets Manager, KMS for at-rest encryption.
> - **Reliability:** ECS Fargate across 3 AZs, RDS Aurora Multi-AZ, DLQ on SQS.
> - **Performance:** ElastiCache Redis for hot product data, CloudFront for images.
> - **Cost:** Fargate Spot for non-critical workers, S3 Intelligent-Tiering for old order receipts.
> - **Sustainability:** Graviton-based Fargate tasks (`runtimePlatform.cpuArchitecture: ARM64`).

## Gotchas

- It's a **framework**, not a certification — no auditor will sign a paper saying you're "Well-Architected."
- Reviews go stale. **Re-run every 6 months** or after big architecture changes.
- The default questions are generic. Add your own for domain-specific risks.

## Related

- [Shared Responsibility](./02-shared-responsibility.md)
- [Pricing Models](./05-pricing-models.md)
