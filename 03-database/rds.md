# RDS — Relational Database Service

**TL;DR** — Managed relational databases: Postgres, MySQL, MariaDB, Oracle, SQL Server. AWS does patching, backups, Multi-AZ failover. You manage schema and queries.

## What it is

You pick an engine + version, instance size, storage. AWS provisions an EC2 + EBS combo running the DB. You connect via a regular DB endpoint and use psql/MySQL clients as normal.

## Engines

- **PostgreSQL** — most loved by devs. Many extensions (PostGIS, pgvector).
- **MySQL** — battle-tested, ubiquitous.
- **MariaDB** — MySQL fork.
- **Oracle** — Bring Your Own License (BYOL) or License Included.
- **SQL Server** — Express/Web/Standard/Enterprise editions.
- **Aurora** — see [aurora.md](./aurora.md); separate but closely related.

## Key concepts

- **DB instance** — the VM running the engine.
- **DB cluster** — Aurora has clusters; regular RDS has standalone instances.
- **Multi-AZ** — synchronous standby in another AZ for HA. ~2x cost.
- **Read replicas** — async replicas for read scaling (up to 15 per source for Postgres/MySQL).
- **Parameter group** — engine config (think `postgresql.conf`).
- **Option group** — engine extensions (e.g. SQL Server Mirroring).
- **Subnet group** — which subnets the DB lives in.
- **Snapshot** — point-in-time backup, stored in S3.
- **Automated backup** — daily snapshot + WAL/binlog → PITR (point-in-time restore).
- **Performance Insights** — built-in query analysis.

## Real-world example

> ShareDeal's orders DB:
> - PostgreSQL 16 on `db.t4g.large` (Graviton, 2 vCPU, 8 GB RAM).
> - Multi-AZ enabled — instant failover on hardware issues.
> - 1 read replica for analytics queries.
> - Backups retained 30 days, snapshots copied to another region.

## Usage

### Create via CLI

```bash
aws rds create-db-instance \
  --db-instance-identifier sharedeal-prod \
  --engine postgres --engine-version 16.4 \
  --db-instance-class db.t4g.large \
  --allocated-storage 100 --storage-type gp3 --storage-encrypted \
  --master-username sdadmin --master-user-password 'change-me!' \
  --vpc-security-group-ids sg-0123 --db-subnet-group-name private-subnets \
  --multi-az \
  --backup-retention-period 30 \
  --enable-performance-insights \
  --deletion-protection
```

### Connect

```bash
psql "host=sharedeal-prod.cluster-xyz.ap-south-1.rds.amazonaws.com port=5432 dbname=postgres user=sdadmin sslmode=require"
```

### Create read replica

```bash
aws rds create-db-instance-read-replica \
  --db-instance-identifier sharedeal-prod-read-1 \
  --source-db-instance-identifier sharedeal-prod
```

### Promote replica (DR scenario)

```bash
aws rds promote-read-replica --db-instance-identifier sharedeal-prod-read-1
```

### Snapshot + restore

```bash
aws rds create-db-snapshot --db-instance-identifier sharedeal-prod --db-snapshot-identifier sharedeal-2026-05-18

aws rds restore-db-instance-from-db-snapshot \
  --db-instance-identifier sharedeal-restored \
  --db-snapshot-identifier sharedeal-2026-05-18
```

### Point-in-time restore (PITR)

```bash
aws rds restore-db-instance-to-point-in-time \
  --source-db-instance-identifier sharedeal-prod \
  --target-db-instance-identifier sharedeal-replay \
  --restore-time 2026-05-18T14:30:00Z
```

## Engine versions & upgrades

- Minor versions auto-applied during maintenance window (configurable).
- Major versions (16 → 17) you trigger manually after testing on a replica or snapshot clone.

## Instance classes worth knowing

- `db.t4g.*` — burstable Graviton, cheap, good for dev/small prod.
- `db.m6g.*` / `db.m7g.*` — general purpose Graviton.
- `db.r7g.*` — memory-optimized for big working sets.
- `db.x2g.*` — extreme memory (SAP, etc.).

## Pricing

- **Instance hours** — like EC2 but ~20% more (managed surcharge).
- **Storage** — $0.115/GB-mo gp3 + IOPS above baseline.
- **Multi-AZ** — 2x instance cost + 2x storage.
- **Backups** — free up to allocated storage size; beyond is $0.095/GB-mo.
- **Data transfer** — same rules as EC2.

A small Multi-AZ Postgres `db.t4g.medium` ≈ **$90-130/mo**.

## Security

- **In VPC**, in a private subnet. Don't expose RDS publicly.
- **IAM database authentication** — short-lived tokens instead of passwords (Postgres/MySQL).
- **Secrets Manager rotation** — auto-rotate master password.
- **SSL/TLS** — `rds-ca-rsa2048-g1` certificate. Always use `sslmode=require` or stricter.
- **Encryption at rest** — KMS, free.

## When NOT to use RDS

- Need horizontal write scaling → DynamoDB, Aurora Limitless, Spanner-like.
- Want serverless auto-scaling → **Aurora Serverless v2**.
- Tiny budget → ElastiCache + S3 might cover your needs.
- Lots of writes per second → Aurora handles 5x more than RDS.

## Gotchas

- **No SSH into the instance.** AWS manages the OS.
- **Free tier**: 750 hr/mo of `db.t2.micro` / `db.t3.micro` / `db.t4g.micro` for 12 months — single AZ only.
- **Multi-AZ standby is invisible.** You can't read from it. Use a read replica instead.
- **Storage auto-grow** is per-engine — enable it but set a max.
- **Max connections** is small at small instance sizes (~100). Use **RDS Proxy** to multiplex.
- **Stopping an RDS instance** stops billing for compute, but only for 7 days, then AWS restarts it.
- **Major version upgrades can take an hour+** depending on size. Test first.

## Related

- [Aurora](./aurora.md) — AWS-native MySQL/Postgres-compatible
- [RDS Proxy](#) — connection pooling for Lambda
- [DMS](../12-migration/dms.md) — migration to/from RDS
- [Secrets Manager](../05-security-iam/secrets-manager.md) — store DB creds
