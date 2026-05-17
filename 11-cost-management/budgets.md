# Budgets

**TL;DR** — Set monthly cost/usage targets; get alerts at thresholds (e.g., 80% of $500). Optional Budget Actions can stop/reduce resources automatically.

## What it is

A simple service for cost/usage/reservation budgets with alerting. **Always set one up on a new account** — it's the cheapest insurance.

## Budget types

- **Cost budget** — $ spent.
- **Usage budget** — units consumed (e.g., 1000 EC2 hours).
- **RI / Savings Plan budget** — utilization or coverage.

## Key concepts

- **Threshold:** 50%, 80%, 100%, 120%, etc.
- **Actual vs Forecasted** — alert if forecast > threshold even before actual.
- **Action** — when crossed, run an IAM policy change, EC2/RDS stop, or SCP.

## Real-world example

> Personal sandbox account:
> - Budget: $25/mo.
> - Alerts at 50%, 80%, 100% via email.
> - Budget Action at 100%: apply an SCP that denies new EC2 launches.

## Usage

```bash
aws budgets create-budget \
  --account-id 123456789012 \
  --budget '{
    "BudgetName":"monthly-25",
    "BudgetLimit":{"Amount":"25","Unit":"USD"},
    "TimeUnit":"MONTHLY",
    "BudgetType":"COST"
  }' \
  --notifications-with-subscribers '[{
    "Notification":{"NotificationType":"ACTUAL","ComparisonOperator":"GREATER_THAN","Threshold":80},
    "Subscribers":[{"SubscriptionType":"EMAIL","Address":"sharedealnow@gmail.com"}]
  }]'
```

## Pricing

- **First 2 budgets free.**
- **$0.02 per day per additional budget.**
- **Budget Actions:** $0.10 each invocation.

## Gotchas

- **Budgets don't stop spending** by default — they only notify. Configure Budget Actions for hard control.
- **Forecast accuracy is low early in the month.**
- **Currency** — set USD for predictability; AWS converts.
- **Tags** — combine with cost-allocation tags for per-project budgets.

## Related

- [Cost Explorer](./cost-explorer.md)
- [Organizations](./organizations.md) — for SCPs
