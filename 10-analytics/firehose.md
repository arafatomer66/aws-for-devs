# Kinesis Data Firehose

**TL;DR** — Easy "stream → S3 / Redshift / OpenSearch / Splunk / HTTP" delivery. Batches, compresses, encrypts, retries. No consumer code.

## What it is

A managed loader: you push records to a Firehose stream; Firehose buffers (by time + size) and writes to your destination automatically. Optional Lambda transform stage in the middle.

## Why it exists

If your goal is "drop stream into S3 as Parquet," you don't want to write a Kinesis Data Streams consumer + Spark partitioner + commit logic. Firehose does it for you.

## Destinations

- **S3** — common; Parquet/ORC conversion built-in (via Glue Schema).
- **Redshift** (via S3 + COPY).
- **OpenSearch.**
- **Splunk.**
- **Generic HTTP endpoint** (Datadog, New Relic, etc.).

## Key concepts

- **Delivery stream** — the pipeline.
- **Buffer hints** — size (1-128 MB) + time (60-900 s). Smaller = lower latency, more S3 files.
- **Transform** — invoke Lambda on each batch (parse, enrich, drop).
- **Format conversion** — Firehose can convert JSON → Parquet using a Glue table schema.
- **Dynamic partitioning** — partition output by record fields (`/year=2026/month=05/`).
- **Source:** Direct PUT, Kinesis Data Stream, MSK.

## Real-world example

> ShareDeal clickstream:
> - Mobile app sends events → API Gateway → Lambda → Firehose `Direct PUT`.
> - Firehose buffers 60 s or 5 MB → converts to Parquet → writes to `s3://clickstream/year=YYYY/month=MM/day=DD/`.
> - Athena queries the partitioned data; QuickSight visualizes.

## Usage

```bash
aws firehose create-delivery-stream \
  --delivery-stream-name clickstream \
  --delivery-stream-type DirectPut \
  --extended-s3-destination-configuration '{
    "RoleARN":"arn:aws:iam::..:role/FirehoseRole",
    "BucketARN":"arn:aws:s3:::clickstream",
    "BufferingHints":{"SizeInMBs":5,"IntervalInSeconds":60},
    "CompressionFormat":"GZIP",
    "Prefix":"events/year=!{timestamp:yyyy}/month=!{timestamp:MM}/day=!{timestamp:dd}/",
    "ErrorOutputPrefix":"errors/"
  }'
```

### Put a record (Node)

```js
import { FirehoseClient, PutRecordCommand } from "@aws-sdk/client-firehose";
const fh = new FirehoseClient({ region: "ap-south-1" });
await fh.send(new PutRecordCommand({
  DeliveryStreamName: "clickstream",
  Record: { Data: Buffer.from(JSON.stringify({ userId, event, ts: Date.now() }) + "\n") },
}));
```

Note the trailing newline — required if downstream reads line-delimited JSON.

## Pricing

- **$0.029 per GB ingested** (first 500 TB/mo); cheaper above.
- **Format conversion to Parquet:** $0.018/GB.
- **VPC delivery:** small extra.

## Kinesis Data Streams vs Firehose

| | Kinesis Data Streams | Firehose |
|---|---|---|
| Consumers | Many, replay | One destination |
| Latency | Sub-second | ~60s minimum |
| Order | Per shard | Best-effort |
| Use | Build streaming apps | Load to storage/search |

Often **paired**: Producer → Kinesis Data Streams → both real-time consumer (Lambda) AND Firehose → S3 (data lake).

## Gotchas

- **Newline-delimited JSON** if the downstream expects it.
- **Buffer hints define minimum latency.**
- **Lambda transform max 6 MB output per record.**
- **Dynamic partitioning has limits** (max 500 active partitions per stream).
- **Errors go to an error S3 prefix** — monitor it.

## Related

- [Kinesis Data Streams](../06-messaging-integration/kinesis.md)
- [Athena](./athena.md)
- [OpenSearch](./opensearch.md)
- [Redshift](../03-database/redshift.md)
