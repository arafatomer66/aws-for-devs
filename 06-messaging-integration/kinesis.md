# Kinesis Data Streams

**TL;DR** — Real-time streaming pipeline. Like Kafka, simpler ops. Producers write records to shards; consumers read in order, can replay. Sub-second latency.

## What it is

A managed log-based stream. Records (max 1 MB) get written to shards (partitions). Consumers read in order. Records stay 24h-365d (retention). Multiple consumers can read independently (each tracks its own position).

## Why it exists

When you need:
- Ordered stream of events (per-shard order).
- Multiple independent consumers replaying.
- Sub-second end-to-end latency.
- High throughput (millions of events/sec).

SQS is queue (one-shot consume). Kinesis is stream (replayable, ordered, fan-out).

## Kinesis family

- **Kinesis Data Streams** — the core stream service.
- **Kinesis Data Firehose** — load streams into S3/Redshift/OpenSearch/Splunk. See [analytics/firehose.md](../10-analytics/firehose.md).
- **Kinesis Data Analytics for Apache Flink** — managed Flink on streams.
- **Kinesis Video Streams** — stream video.

## Key concepts

- **Stream** — the top-level resource.
- **Shard** — a partition. Each shard handles 1 MB/s write + 2 MB/s read.
- **Partition key** — determines which shard a record goes to (`hash(partitionKey) % shards`).
- **Capacity modes:**
  - **On-Demand** — auto-scales shards. Pay per usage.
  - **Provisioned** — you set N shards.
- **Retention** — 24h default, up to 365d.
- **Enhanced fan-out (EFO)** — dedicated 2 MB/s per consumer per shard (vs shared 2 MB/s).

## Real-world example

> Clickstream pipeline:
> - Web/mobile → Kinesis Data Stream `clicks` (32 shards).
> - Consumer 1: Lambda → DynamoDB (real-time user activity).
> - Consumer 2: Firehose → S3 (data lake).
> - Consumer 3: Flink → real-time fraud detection.
>
> All read independently from the same stream.

## Usage

### Create

```bash
aws kinesis create-stream --stream-name clicks --shard-count 4
# Or on-demand
aws kinesis create-stream --stream-name clicks --stream-mode-details StreamMode=ON_DEMAND
```

### Put records (Node)

```js
import { KinesisClient, PutRecordCommand } from "@aws-sdk/client-kinesis";
const ks = new KinesisClient({ region: "ap-south-1" });

await ks.send(new PutRecordCommand({
  StreamName: "clicks",
  Data: Buffer.from(JSON.stringify({ userId: "u_42", page: "/product/999" })),
  PartitionKey: "u_42",   // keeps same-user events on the same shard
}));
```

Use `PutRecords` to batch up to 500 records.

### Consume via Lambda

Easiest. Lambda polls shards, invokes function per batch.
```ts
fn.addEventSource(new KinesisEventSource(stream, {
  batchSize: 100, startingPosition: lambda.StartingPosition.LATEST,
  parallelizationFactor: 4,  // concurrent invokes per shard
  reportBatchItemFailures: true,
}));
```

### Consume via KCL (Java/Python)

Kinesis Client Library handles checkpoints, shard rebalancing, lease management. Heavy but robust for self-managed consumers.

## Pricing

- **Provisioned:** $0.015/shard-hr (~$11/mo per shard) + $0.014 per million PUT payload units.
- **On-Demand:** $0.04/GB written + $0.04/GB read.
- **Extended retention beyond 24h:** $0.02/shard-hr.
- **Enhanced fan-out:** $0.015/consumer-shard-hr.

## Kinesis vs Kafka (MSK) vs SQS vs EventBridge

| | Kinesis | MSK (Kafka) | SQS | EventBridge |
|---|---|---|---|---|
| Order | Per shard | Per partition | FIFO opt | No |
| Replay | Yes (retention) | Yes (retention) | No | Archive/Replay only |
| Throughput | Millions/sec | Millions/sec | High | Moderate |
| Ops effort | Low | Medium | Lowest | Lowest |
| Best for | Streams, replay | Existing Kafka ecosystem | Async tasks | Event-driven routing |

## Gotchas

- **Hot shard** — uneven partition keys → one shard hammered. Pick keys with good distribution.
- **Shard splits/merges** for resizing — slow, plan ahead. On-Demand mode dodges this.
- **Consumer lag** — monitor `IteratorAge`. If it grows, increase parallelism or shards.
- **1 MB record max.** Use S3 pointers for bigger.
- **No DLQ natively.** Use Lambda destinations / KCL error handling.
- **Records expire after retention** — don't treat as long-term storage.

## Related

- [Firehose](../10-analytics/firehose.md)
- [MSK](./msk.md)
- [Lambda](../01-compute/lambda.md)
