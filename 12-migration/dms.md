# DMS — Database Migration Service

**TL;DR** — Migrate databases to AWS with minimal downtime. Supports Postgres / MySQL / Oracle / SQL Server / MongoDB ↔ RDS / Aurora / DocumentDB / S3 / Kinesis / etc. Full load + CDC.

## What it is

A managed migration service. You define source + target endpoints, a replication instance (or DMS Serverless), and a task. DMS does:
- **Full load** — copy existing data.
- **CDC (change data capture)** — apply ongoing writes from the source so you can cut over with seconds of downtime.

Also useful for **ongoing replication** (not just one-shot migration).

## Key concepts

- **Endpoint** — source or target DB.
- **Replication instance** — EC2 that runs the migration. Or DMS Serverless (auto-sized).
- **Task** — what to migrate (which schemas/tables), mode (full / CDC / both).
- **Schema Conversion Tool (SCT)** — separate tool to translate schemas across heterogeneous engines (Oracle → Postgres).
- **Validation** — DMS can compare source vs target row counts/values.

## Real-world example

> Postgres on-prem → Aurora Postgres in `ap-south-1`:
> 1. DMS replication instance in VPC, reachable via Site-to-Site VPN.
> 2. Source endpoint = on-prem Postgres, target = Aurora.
> 3. Task: full load + CDC.
> 4. Wait for full load to finish; CDC continues syncing writes.
> 5. Cut over: stop writes to source, wait for CDC lag to hit 0, switch app to Aurora.
> 6. Downtime: < 1 minute.

## Usage

```bash
aws dms create-replication-instance --replication-instance-identifier dms-main \
  --replication-instance-class dms.t3.medium --allocated-storage 50 \
  --vpc-security-group-ids sg-... --replication-subnet-group-identifier dms-subnets

aws dms create-endpoint --endpoint-identifier src-pg --endpoint-type source \
  --engine-name postgres --server-name 10.0.0.5 --port 5432 \
  --username dms_user --password 'dms-pw' --database-name app

aws dms create-endpoint --endpoint-identifier tgt-aurora --endpoint-type target \
  --engine-name aurora-postgresql --server-name aurora.cluster-xxxx... --port 5432 \
  --username dms_admin --password 'dms-pw' --database-name app

aws dms create-replication-task --replication-task-identifier migrate-app \
  --source-endpoint-arn arn:aws:dms:..:endpoint/src-pg \
  --target-endpoint-arn arn:aws:dms:..:endpoint/tgt-aurora \
  --replication-instance-arn arn:aws:dms:..:rep/dms-main \
  --migration-type full-load-and-cdc \
  --table-mappings file://table-mappings.json
```

## Pricing

- **Replication instance hours:** standard EC2 pricing.
- **DMS Serverless:** $0.20/DCU-hr (DMS Capacity Unit).
- **No additional DMS fee.**

## DMS targets — beyond databases

DMS can also write to S3 (CSV/Parquet), Kinesis, OpenSearch — useful for building real-time data lakes from existing DBs.

## Gotchas

- **CDC requires source-side config** (Postgres: `logical_decoding`, MySQL: binlog row mode, Oracle: archive log on).
- **Large LOB columns** — separate LOB mode, slower.
- **Heterogeneous migrations** need SCT for schema translation; queries/views likely need rewrites.
- **DDL changes during migration** can break tasks — pause schema changes during cutover window.
- **Validation finds bugs** — turn it on.

## Related

- [RDS](../03-database/rds.md)
- [Aurora](../03-database/aurora.md)
- [DataSync](./datasync.md) — for files, not DBs
