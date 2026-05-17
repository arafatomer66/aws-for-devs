# Compute Optimizer

**TL;DR** — ML-based right-sizing recommendations for EC2, EBS, Lambda, ECS on Fargate, Auto Scaling Groups, RDS. Free. Run it monthly.

## What it is

Compute Optimizer ingests CloudWatch metrics and recommends smaller (or sometimes larger) configurations. Outputs include estimated cost savings and performance risk.

## What it covers

- **EC2** — instance type / size recommendations.
- **Auto Scaling Groups** — recommended mix.
- **EBS** — gp2 → gp3 (almost always cheaper + faster).
- **Lambda** — memory tuning (which affects CPU + cost).
- **ECS Fargate tasks** — CPU/RAM right-sizing.
- **RDS** — instance class right-sizing.
- **Commercial software** licensing for EC2.

## Real-world example

> A fleet of 30 `m5.large` EC2s. Compute Optimizer says:
> - 18 are underutilized → downsize to `m6g.medium` (Graviton).
> - 2 are over-utilized → upsize to `m5.xlarge`.
>
> Estimated savings: ~$400/mo.

## Usage

Console: AWS Compute Optimizer dashboard. Or API:
```bash
aws compute-optimizer get-ec2-instance-recommendations
aws compute-optimizer get-lambda-function-recommendations
```

Output: per-resource recommendation + finding (`UNDER_PROVISIONED` / `OVER_PROVISIONED` / `OPTIMIZED`).

## Pricing

- **Free** for basic recommendations.
- **Enhanced infrastructure metrics** (longer history, finer): $0.0003 per resource per hour.

## Gotchas

- **Needs 14+ days of data** for confident recommendations.
- **Doesn't account for workload patterns it can't see** (memory pressure if you don't run the CW agent).
- **Lambda memory tuning** can both lower latency and cost — try recommendations in a test env first.

## Related

- [Cost Explorer](./cost-explorer.md)
- [Trusted Advisor](./trusted-advisor.md)
