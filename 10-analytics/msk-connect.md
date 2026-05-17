# MSK Connect

**TL;DR** — Managed Kafka Connect for MSK. Run source/sink connectors (S3, OpenSearch, JDBC, etc.) without operating Connect clusters.

## What it is

A managed Kafka Connect runtime. Submit a connector plugin + config; MSK Connect runs it as workers, autoscales, restarts on failure.

## Key concepts

- **Custom plugin** — JAR for the connector (Debezium, JDBC, S3, Elasticsearch, etc.).
- **Worker configuration** — connect runtime tuning.
- **Connector** — actual running instance (source or sink).
- **MCU (MSK Connect Unit)** — 1 vCPU + 4 GB. You set min/max.

## Real-world example

> Streaming order events from Postgres to OpenSearch:
> - Source connector: Debezium for Postgres → MSK topic.
> - Sink connector: OpenSearch sink → indexes orders for search.
> - All via MSK Connect. No EC2 to babysit.

## Pricing

- ~$0.11 per MCU-hr (similar to MSK brokers).
- Plus the underlying MSK cluster cost.

## Gotchas

- **Plugin compatibility** — some open-source connectors need patching for MSK Connect.
- **Limited region availability** initially.
- **Connect concepts (offsets, status, config topics)** still apply — MSK Connect uses Kafka topics for state.

## Related

- [MSK](../06-messaging-integration/msk.md)
- [Firehose](./firehose.md)
- [Glue](./glue.md)
