# Keyspaces (for Apache Cassandra)

**TL;DR** — Managed Cassandra-compatible service. Cassandra CQL API, AWS-style serverless + auto-scaling. Drop-in for many Cassandra workloads.

## What it is

A serverless implementation of the Cassandra Query Language (CQL) API. Same drivers, same queries, but no nodes to manage. AWS handles compaction, repair, hinted handoff, scaling.

## Why it exists

Cassandra at scale needs careful operational tuning. Keyspaces strips that away — you get CQL with DynamoDB-style operations.

## Key concepts

- **Keyspace** = database.
- **Table** with partition key + clustering columns (standard Cassandra schema).
- **On-Demand** or **Provisioned** capacity (like DynamoDB).
- **CQL drivers** — DataStax drivers in any language work.
- TLS authentication via AWS SigV4 plugin or service-specific credentials.

## Real-world example

> A team migrating from on-prem Cassandra:
> - `cqlsh` connects with SigV4 plugin.
> - All `CREATE TABLE` statements work.
> - Schema migration tool runs as-is.
> - Workload runs without app changes; ops effort dropped to zero.

## Usage

```cql
CREATE KEYSPACE sd WITH REPLICATION = { 'class':'SingleRegionStrategy' };

CREATE TABLE sd.events (
  user_id text,
  event_time timestamp,
  type text,
  payload text,
  PRIMARY KEY ((user_id), event_time)
) WITH CLUSTERING ORDER BY (event_time DESC);

INSERT INTO sd.events (user_id, event_time, type, payload) VALUES ('u_42', toTimestamp(now()), 'login', '{}');
```

Connect from app — same as standard Cassandra, with the AWS SigV4 plugin for auth.

## Pricing

- **Storage:** $0.30/GB-mo.
- **Writes:** $1.45 per million (1 KB).
- **Reads:** $0.29 per million.

## When Keyspaces vs DynamoDB

- **Keyspaces** — you have existing Cassandra code/skills.
- **DynamoDB** — starting fresh, want first-class AWS integration.

## Gotchas

- **Not 100% Cassandra parity** — some features (lightweight transactions, secondary indexes) limited/missing.
- **Per-request costs add up** at high QPS — provisioned mode for steady high traffic.
- **Multi-region** is configurable but check current capabilities vs OSS Cassandra.

## Related

- [DynamoDB](./dynamodb.md)
