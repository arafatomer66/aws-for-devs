# SQS — Simple Queue Service

**TL;DR** — Managed message queue. Producers send messages, consumers poll. At-least-once delivery. Decouples services. Cheap, durable, scales infinitely.

## What it is

A pull-based queue. A producer sends messages to a queue; consumers (Lambda, EC2, ECS) poll for messages and process them. AWS holds messages durably until processed (and `DeleteMessage` is called) or the retention period expires (1 min – 14 days).

## Queue types

- **Standard** — best-effort ordering, **at-least-once** delivery, unlimited throughput. Default.
- **FIFO** — strict ordering per Message Group ID, exactly-once processing, 3000 msg/s (with batching) per group.

## Key concepts

- **Producer / Consumer** — sender / receiver.
- **Message** — up to 256 KB (use S3 for larger via Extended Client).
- **Visibility timeout** — when a consumer receives a message, it's "invisible" for N seconds while processing. If not deleted in time, it reappears.
- **Retention** — how long unprocessed messages live (default 4 days, max 14 days).
- **Dead Letter Queue (DLQ)** — after N receive failures, message moves here for inspection.
- **Long polling** — `WaitTimeSeconds = 20` to avoid empty-poll cost.
- **Batch** — send/receive up to 10 messages per call.
- **Message Group ID** (FIFO only) — messages with same group ID are ordered.
- **Deduplication ID** (FIFO only) — exactly-once within a 5-min dedup window.

## Real-world example

> ShareDeal order pipeline:
> - User clicks "Place Order" → API writes to DB → publishes to SQS `orders-to-fulfill`.
> - Worker Lambda (or ECS task) pulls from queue → emails confirmation, notifies warehouse, charges card.
> - If a step fails 5 times → DLQ → on-call investigates.
>
> If the worker crashes mid-message, the visibility timeout expires, the message reappears, another worker tries.

## Usage

### Create

```bash
aws sqs create-queue --queue-name orders-to-fulfill \
  --attributes '{"VisibilityTimeout":"60","MessageRetentionPeriod":"345600"}'

# FIFO queue (.fifo suffix required)
aws sqs create-queue --queue-name orders.fifo \
  --attributes '{"FifoQueue":"true","ContentBasedDeduplication":"true"}'
```

### Send

```bash
aws sqs send-message --queue-url <url> \
  --message-body '{"orderId":"ord_42","userId":"u_1"}' \
  --message-attributes '{"type":{"DataType":"String","StringValue":"order.placed"}}'
```

### Receive + delete (Node)

```js
import { SQSClient, ReceiveMessageCommand, DeleteMessageCommand } from "@aws-sdk/client-sqs";
const sqs = new SQSClient({ region: "ap-south-1" });

const { Messages } = await sqs.send(new ReceiveMessageCommand({
  QueueUrl: queueUrl,
  MaxNumberOfMessages: 10,
  WaitTimeSeconds: 20,  // long polling
  VisibilityTimeout: 60,
}));

for (const msg of Messages ?? []) {
  try {
    await processOrder(JSON.parse(msg.Body));
    await sqs.send(new DeleteMessageCommand({ QueueUrl: queueUrl, ReceiptHandle: msg.ReceiptHandle }));
  } catch (e) {
    // don't delete → it'll reappear after visibility timeout
  }
}
```

### Lambda as consumer (no polling code)

Wire SQS → Lambda event source. Lambda polls for you, scales horizontally, deletes on success, returns to queue on failure.

```ts
// CDK
const fn = new lambda.Function(this, "OrderWorker", { ... });
fn.addEventSource(new SqsEventSource(queue, {
  batchSize: 10,
  reportBatchItemFailures: true,  // partial batch failure
}));
```

### Send with DLQ

```bash
aws sqs set-queue-attributes --queue-url <main-q-url> --attributes '{
  "RedrivePolicy": "{\"deadLetterTargetArn\":\"arn:aws:sqs:..:queue-name-dlq\",\"maxReceiveCount\":5}"
}'
```

## Pricing

- **First 1M requests/mo: free** (always free tier).
- **Beyond:** $0.40 per million (Standard), $0.50 (FIFO).
- **Long polling** doesn't cost extra and reduces empty-receive count.

## When SQS vs SNS vs EventBridge vs MSK/Kinesis

| | SQS | SNS | EventBridge | Kinesis/MSK |
|---|---|---|---|---|
| Model | Queue (pull) | Topic (push, fan-out) | Event bus (rules) | Stream (replay, order) |
| Consumers | One per msg | Many (subscribers) | Many (routing rules) | Many (shard-ordered) |
| Order | FIFO opt | No | No | Yes (per shard) |
| Replay | No | No | Limited (Archive/Replay) | Yes (TTL) |
| Best for | Async jobs | Pub/sub fan-out | Event-driven w/ rules | Streaming pipelines |

## Gotchas

- **At-least-once delivery on Standard.** Make consumers **idempotent**.
- **Visibility timeout > processing time.** Otherwise duplicate processing.
- **DLQ is mandatory for prod.** Without it, poison messages loop forever (or expire silently).
- **FIFO throughput limited per Message Group ID** — fan messages across many groups for parallelism.
- **256 KB max message size.** For larger, use S3 Extended Client (puts payload in S3, message has pointer).
- **Empty receives still cost money** unless using long polling.
- **No native priority queues** — use separate queues + weighted polling.

## Related

- [SNS](./sns.md)
- [EventBridge](./eventbridge.md)
- [Lambda](../01-compute/lambda.md)
- [Step Functions](./step-functions.md)
