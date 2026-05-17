# EventBridge

**TL;DR** — Event bus with rich routing rules. Events from AWS services, your apps, and SaaS sources route to many targets. The modern way to do event-driven AWS.

## What it is

EventBridge is a managed event bus. You put events (JSON) on a bus; rules pattern-match against the events and route them to targets (Lambda, SQS, SNS, Step Functions, ECS tasks, API destinations, Kinesis, etc.).

Built on (and largely replaces) **CloudWatch Events**.

## Three event sources

1. **AWS services** — most fire events on a default bus (S3 PutObject, EC2 state change, GuardDuty findings, CodePipeline transitions, etc.).
2. **Your own apps** — `PutEvents` API to a custom bus.
3. **SaaS partners** — Datadog, Auth0, Shopify, Zendesk, MongoDB Atlas, etc. via Partner event sources.

## Key concepts

- **Event bus** — default, custom, or partner.
- **Rule** — event pattern (JSON match) + targets.
- **Event pattern** — JSON describing what to match (source, detail-type, detail.field, prefix, anything-but, numeric ranges).
- **Targets** — Lambda, SQS, SNS, Step Functions, ECS task, Kinesis stream, Firehose, API destination, other bus, Batch job, Glue workflow, etc. (~30+).
- **Input transformer** — modify the event before sending to target.
- **Archive + Replay** — store events; replay to re-trigger rules.
- **EventBridge Scheduler** — cron jobs as a separate (newer) feature.
- **EventBridge Pipes** — source → filter → enrich → target (DynamoDB Stream → Lambda → SQS, etc.) without writing glue Lambdas.

## Real-world example

> ShareDeal architecture:
> - Custom bus `selefe-events`.
> - App writes events: `order.placed`, `payment.received`, `delivery.completed`.
> - Rules:
>   - `order.placed` → Step Function `OrderFulfillment`.
>   - `payment.received` AND `amount > 10000` → SNS `high-value-alerts`.
>   - All events → Firehose → S3 (audit lake).
> - Scheduler: daily `reconcile-payments` Lambda at 02:00 UTC.

## Usage

### Custom bus + rule

```bash
aws events create-event-bus --name selefe-events

aws events put-rule --name high-value-orders \
  --event-bus-name selefe-events \
  --event-pattern '{
    "source": ["sd.orders"],
    "detail-type": ["order.placed"],
    "detail": { "total": [{"numeric": [">=", 10000]}] }
  }'

aws events put-targets --rule high-value-orders --event-bus-name selefe-events \
  --targets 'Id=1,Arn=arn:aws:sns:ap-south-1:..:high-value-alerts'
```

### Put events

```bash
aws events put-events --entries '[{
  "EventBusName": "selefe-events",
  "Source": "sd.orders",
  "DetailType": "order.placed",
  "Detail": "{\"orderId\":\"ord_42\",\"total\":12500}"
}]'
```

### Node SDK

```js
import { EventBridgeClient, PutEventsCommand } from "@aws-sdk/client-eventbridge";
const eb = new EventBridgeClient({ region: "ap-south-1" });

await eb.send(new PutEventsCommand({
  Entries: [{
    EventBusName: "selefe-events",
    Source: "sd.orders",
    DetailType: "order.placed",
    Detail: JSON.stringify({ orderId: "ord_42", total: 12500 }),
  }],
}));
```

### Scheduler (cron)

```bash
aws scheduler create-schedule \
  --name daily-reconcile \
  --schedule-expression "cron(0 2 * * ? *)" \
  --target '{"Arn":"arn:aws:lambda:ap-south-1:..:function:reconcile","RoleArn":"arn:aws:iam::..:role/scheduler-role"}' \
  --flexible-time-window '{"Mode":"OFF"}'
```

(EventBridge Scheduler is the recommended cron replacement vs old `events put-rule --schedule-expression`.)

### Pipes example

DynamoDB Stream → filter → Lambda:
```bash
aws pipes create-pipe --name new-users-pipe \
  --source arn:aws:dynamodb:..:table/users/stream/... \
  --target arn:aws:lambda:..:function:onboard-user \
  --role-arn arn:aws:iam::..:role/pipes-role \
  --source-parameters '{"DynamoDBStreamParameters":{"StartingPosition":"LATEST"},"FilterCriteria":{"Filters":[{"Pattern":"{\"eventName\":[\"INSERT\"]}"}]}}'
```

## Pricing

- **Default + AWS service events:** free.
- **Custom + partner events:** $1.00 per million events published.
- **Scheduler:** $1.00 per million invocations.
- **Pipes:** $0.40 per million + small enrichment cost.
- **Archive:** $0.10/GB-mo + processing.

## EventBridge vs SNS

| | EventBridge | SNS |
|---|---|---|
| Routing rules | Rich JSON pattern, attribute filters | Simple filter on attributes |
| Targets | 30+, deep AWS integrations | Few (SQS, Lambda, HTTP, email, SMS) |
| Throughput | High | Higher |
| Per-message cost | $1/M | $0.50/M |
| Use | Event-driven workflows, SaaS integration | Pub/sub fan-out, alerts |

Default to EventBridge for new architectures. SNS still wins for raw fan-out to many subscribers at scale, or for SMS/email.

## Gotchas

- **Best-effort delivery** to most targets. Configure target retry + DLQ.
- **Schema registry** is a separate feature (helps with strongly typed event consumers).
- **Custom bus events count toward the per-million bill** even if rules don't match.
- **Cross-account** routing supported but needs explicit bus policy.
- **Event size limit 256 KB.**

## Related

- [SNS](./sns.md)
- [SQS](./sqs.md)
- [Step Functions](./step-functions.md)
- [Lambda](../01-compute/lambda.md)
