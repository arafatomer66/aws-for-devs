# SNS — Simple Notification Service

**TL;DR** — Pub/sub topics. Publish once, fan out to many subscribers (SQS, Lambda, HTTP, email, SMS, mobile push). Push-based (vs SQS's pull).

## What it is

A topic-based pub/sub. Publishers send messages to a topic; subscribers (multiple types) get pushed copies. Standard or FIFO.

## Subscriber types

- **SQS queue** — most common pattern (fan-out to queues).
- **Lambda** — invoked per message.
- **HTTP/HTTPS endpoint** — webhooks.
- **Email / Email-JSON** — alerts.
- **SMS** — one-off texts (separate worldwide messaging service for big SMS).
- **Mobile push** — APNs/FCM/GCM/ADM/Baidu, via Platform Applications.
- **Kinesis Data Firehose** — fan messages to S3/Redshift.

## Key concepts

- **Topic** — the named channel.
- **Subscription** — endpoint + protocol.
- **Message filtering** — subscribers receive only messages matching their filter policy (JSON pattern on attributes).
- **FIFO topic** — paired with FIFO SQS queues.
- **Cross-account / cross-region** — supported.

## Real-world example

> User creates an order. SNS topic `order.created`:
> - Subscriber 1: SQS queue `email-jobs` → email worker.
> - Subscriber 2: SQS queue `analytics-events` → analytics consumer.
> - Subscriber 3: Lambda `update-inventory`.
> - Subscriber 4: HTTP webhook to vendor partner.
>
> One publish, four reactions.

## Usage

### Create + publish

```bash
aws sns create-topic --name order-created
# Returns TopicArn

aws sns publish --topic-arn arn:aws:sns:ap-south-1:..:order-created \
  --message '{"orderId":"ord_42"}' \
  --message-attributes '{"region":{"DataType":"String","StringValue":"ap-south-1"}}'
```

### Subscribe an SQS queue

```bash
aws sns subscribe --topic-arn arn:aws:sns:...:order-created \
  --protocol sqs --notification-endpoint arn:aws:sqs:...:email-jobs
```

Also: update the SQS queue policy to allow SNS to write.

### Filter policy

Subscriber only sees orders in their region:
```bash
aws sns set-subscription-attributes \
  --subscription-arn <sub-arn> \
  --attribute-name FilterPolicy \
  --attribute-value '{"region":["ap-south-1"]}'
```

### CDK

```ts
const topic = new sns.Topic(this, "OrderCreated");
topic.addSubscription(new SqsSubscription(emailQueue));
topic.addSubscription(new LambdaSubscription(inventoryFn));
```

## Pricing

- **First 1M publishes free.**
- **Standard:** $0.50 per million.
- **FIFO:** $0.30 per million + $0.01 per 100k bytes.
- **HTTP/HTTPS delivery:** $0.60 per million.
- **Email:** $2 per 100k.
- **SMS:** varies by country (~$0.00645 per US message).

## Gotchas

- **Push-based** — no polling; subscribers receive immediately.
- **No native retry queue** for failed pushes (HTTP/HTTPS). Configure SNS DLQ.
- **SMS pricing varies widely** by destination country. Don't ramp up without checking.
- **Filter policy is on subscription, not topic** — so you can have multiple subscribers with different filters on the same topic.
- **FIFO topics only fan out to FIFO SQS** (not Lambda directly).

## Related

- [SQS](./sqs.md) — paired with SNS for the classic "SNS→SQS fan-out" pattern
- [EventBridge](./eventbridge.md) — newer, richer routing
- [SES](#) — for transactional email
