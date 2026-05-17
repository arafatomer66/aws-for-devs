# Aurora

**TL;DR** — AWS's cloud-native MySQL/PostgreSQL-compatible engine. Same wire protocol, 3-5x faster, auto-scaling storage, 6-way replicated across 3 AZs. Costs more than RDS, scales much further.

## What it is

A re-implementation of MySQL/Postgres where AWS rewrote the storage engine to use a distributed, shared, cloud-native log layer. Up to 128 TB volumes, replicated across 3 AZs, automatic.

Same SQL, same drivers, same admin commands as MySQL/Postgres — just better operational characteristics.

## Why it exists

Open-source MySQL/Postgres weren't designed for the cloud. Aurora delivers:
- Auto-scaling storage (10 GB → 128 TB).
- Faster failover (~30s vs minutes for RDS Multi-AZ).
- Up to 15 low-latency read replicas (1-2ms lag).
- 6-copy storage layer across 3 AZs.
- 3-5x MySQL throughput, 3x Postgres throughput.

## Key concepts

- **Cluster** — primary writer + 0–15 readers + shared storage volume.
- **Writer endpoint** — the single instance that can take writes.
- **Reader endpoint** — round-robins across readers.
- **Cluster endpoint** — alias for the writer.
- **Custom endpoint** — pick a subset of replicas.
- **Aurora Serverless v2** — instances auto-scale 0.5 → 256 ACUs (~2 GB ↔ ~1 TB RAM).
- **Aurora Global Database** — cross-region replication, < 1s lag.
- **Backtrack** (MySQL only) — rewind the DB in-place to a past time.
- **Database Activity Streams** — audit log to Kinesis.
- **Aurora Limitless Database** (2024) — Aurora that shards transparently.
- **Babelfish** — Aurora Postgres that speaks SQL Server's TDS protocol.

## Aurora vs RDS

| | RDS Postgres | Aurora Postgres |
|---|---|---|
| Wire protocol | Postgres | Postgres |
| Storage | EBS, fixed at create (auto-grow opt-in) | Shared, auto-grow up to 128 TB |
| Replication | 1 sync standby (Multi-AZ) + async read replicas | 6 copies in storage layer; up to 15 readers |
| Failover | 1-2 min | ~30s |
| Throughput | Native PG | ~3x native PG |
| Cost | Lower | ~20% higher per instance + storage I/O charges |

**Default to Aurora** for new prod workloads unless you need a feature only in stock Postgres (e.g. specific extension version).

## Real-world example

> Multi-tenant SaaS with 1 TB DB, growing 100 GB/month:
> - Aurora Postgres cluster, writer = `db.r7g.xlarge`, 2 readers across AZs.
> - Storage auto-grows; no DBA babysitting.
> - Reader endpoint serves analytics dashboards without affecting writes.
> - Global Database replica in DR region for sub-second failover.

## Usage

### Create cluster

```bash
aws rds create-db-cluster \
  --db-cluster-identifier sd-cluster \
  --engine aurora-postgresql --engine-version 16.4 \
  --master-username sdadmin --master-user-password 'change-me!' \
  --vpc-security-group-ids sg-0123 --db-subnet-group-name private-subnets \
  --storage-encrypted \
  --backup-retention-period 14

# Add instances to the cluster
aws rds create-db-instance \
  --db-instance-identifier sd-writer \
  --db-cluster-identifier sd-cluster \
  --db-instance-class db.r7g.large \
  --engine aurora-postgresql

aws rds create-db-instance \
  --db-instance-identifier sd-reader-1 \
  --db-cluster-identifier sd-cluster \
  --db-instance-class db.r7g.large \
  --engine aurora-postgresql
```

Connect to:
- Writer endpoint: `sd-cluster.cluster-xyz.ap-south-1.rds.amazonaws.com`
- Reader endpoint: `sd-cluster.cluster-ro-xyz.ap-south-1.rds.amazonaws.com`

### Aurora Serverless v2

```bash
aws rds create-db-cluster \
  --db-cluster-identifier sd-serverless \
  --engine aurora-postgresql --engine-version 16.4 \
  --master-username sdadmin --master-user-password 'change-me!' \
  --serverless-v2-scaling-configuration MinCapacity=0.5,MaxCapacity=32 \
  ...
aws rds create-db-instance \
  --db-instance-identifier sd-serverless-1 \
  --db-cluster-identifier sd-serverless \
  --db-instance-class db.serverless \
  --engine aurora-postgresql
```

ACU = Aurora Capacity Unit. 0.5 ACU = ~1 GB RAM + matching CPU. Bills per second.

## Pricing

- **Instance:** ~$0.29/hr for `db.r7g.large` (~$210/mo per instance).
- **Storage:** $0.10/GB-mo (vs RDS's $0.115).
- **I/O:** $0.20 per million requests (or use **I/O-Optimized** mode: more expensive per instance, free I/O — usually wins if I/O > 25% of bill).
- **Backup:** $0.021/GB-mo beyond cluster size.

A modest 2-instance prod cluster ≈ **$500-700/mo**.

## Aurora Serverless v2 specifics

- Scales **in real-time** down to 0.5 ACU minimum (can scale to 0 in newer versions / preview).
- Best for unpredictable / spiky workloads.
- Slightly higher per-ACU cost than provisioned; cheaper at low utilization.

## Gotchas

- **One writer per cluster** (until Aurora Limitless). All writes serialize there.
- **Aurora I/O charges can sneak up** — `I/O-Optimized` mode is worth checking.
- **Storage is shared** but compute is per-instance — sizing replicas matters for read latency.
- **Backtrack only on MySQL**, not Postgres.
- **Global Database replica is read-only** until promoted.
- **PG extensions** — Aurora supports most but not all (e.g. some PostGIS versions lag).

## Related

- [RDS](./rds.md)
- [DynamoDB](./dynamodb.md) — when relational isn't the right model
- [Database Activity Streams](#)
- [RDS Proxy](#) — connection pooling
