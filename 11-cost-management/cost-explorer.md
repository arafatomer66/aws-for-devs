# Cost Explorer

**TL;DR** — Visualize and filter AWS spend. Group by service, tag, account, region. Forecast future spend. Daily granularity.

## What it is

A reporting UI + API over your AWS billing data. Slice and dice cost/usage by service, account, region, tag, usage type, instance type, purchase option, etc.

## Key concepts

- **Cost** — what you paid.
- **Usage** — units consumed.
- **Group by** — Service, Tag, Account, Region, Usage Type, etc.
- **Filter** — narrow to a subset.
- **Granularity** — daily or hourly (hourly is paid feature).
- **Resource-level cost** — opt-in; lets you see cost per EC2 instance / S3 bucket etc.
- **Forecast** — predicted next-N-day spend.
- **Savings Plans / RI utilization & coverage reports.**

## Setup

1. Activate **Cost Explorer** in Billing Console (one-time, can take 24 h to populate).
2. Activate **Cost Allocation Tags** in Billing → Cost allocation tags. Without this, your `Project=` / `Env=` tags don't show up in Cost Explorer.
3. Enable hourly + resource-level granularity if you need them (paid).

## Real-world example

> "Why did our bill jump $2k this month?"
> - Cost Explorer → Group by Service → Compare to last month.
> - Spot the culprit: NAT Gateway data processing tripled.
> - Drill in: tagged by `Project`, find the project responsible.
> - Action: add S3 VPC endpoint → cut traffic that goes via NAT.

## Use the API for automation

```js
import { CostExplorerClient, GetCostAndUsageCommand } from "@aws-sdk/client-cost-explorer";
const ce = new CostExplorerClient({ region: "us-east-1" });  // CE API is us-east-1 only

const { ResultsByTime } = await ce.send(new GetCostAndUsageCommand({
  TimePeriod: { Start: "2026-05-01", End: "2026-06-01" },
  Granularity: "DAILY",
  Metrics: ["UnblendedCost"],
  GroupBy: [{ Type: "DIMENSION", Key: "SERVICE" }],
}));
```

## Pricing

- **Cost Explorer UI:** free.
- **API requests:** $0.01 per request.
- **Hourly + resource-level granularity:** ~$0.30 per million records.

## Tagging strategy (essential)

Activate these as Cost Allocation Tags:
- `Project`
- `Env`
- `Team` / `Owner`
- `CostCenter` / `Customer`

Tag every resource. Audit untagged resources monthly (Tag Editor / Config).

## Gotchas

- **CE API is `us-east-1` only.**
- **Reports lag ~24 h.**
- **Some costs aren't broken down per resource** (NAT data, taxes).
- **"Blended" vs "Unblended" cost** — unblended is the simple per-resource cost. Use unblended.

## Related

- [Budgets](./budgets.md)
- [Trusted Advisor](./trusted-advisor.md)
- [Compute Optimizer](./compute-optimizer.md)
- [Pricing Models](../00-foundations/05-pricing-models.md)
