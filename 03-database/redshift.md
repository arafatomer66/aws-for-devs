# Redshift

**TL;DR** — AWS's data warehouse. Columnar, MPP (massively parallel processing), petabyte-scale SQL. For analytics, BI, dashboards — not OLTP.

## What it is

A managed data warehouse. Postgres-derived SQL dialect (mostly compatible), columnar storage, distributed across nodes, optimized for "scan a billion rows and aggregate" workloads.

## OLTP vs OLAP

- **OLTP** (Online Transaction Processing) — many small reads/writes (RDS/Aurora/Dynamo). Customer-facing apps.
- **OLAP** (Online Analytical Processing) — few huge analytic queries (Redshift/Snowflake/BigQuery). Dashboards, BI.

Use Redshift for OLAP. Don't run an app on it.

## Architecture

- **Leader node** — coordinator, plans queries.
- **Compute nodes** — store data + execute query slices in parallel.
- **Two deployment modes:**
  - **Provisioned** — pick `ra3.4xlarge` etc., scale by adding nodes.
  - **Serverless** — pay per RPU-second; cluster spins up/down as needed.

## Key concepts

- **Distribution key** — column used to shard data across nodes (`EVEN`, `KEY`, `ALL`).
- **Sort key** — column used to physically order data on disk (skips entire blocks on `WHERE`).
- **Spectrum** — query S3 data directly via Redshift (Redshift Spectrum or Lake Formation tables).
- **Concurrency Scaling** — auto-add transient compute for read spikes.
- **AQUA** — hardware-accelerated cache for hot data.
- **RA3 nodes** — separate compute and storage; pay for what you use of S3-backed storage.
- **Materialized views** — precomputed query results, auto-refreshed.
- **Data Sharing** — share live data across Redshift clusters/accounts without copying.
- **Federated Query** — query RDS/Aurora directly from Redshift.

## Real-world example

> A marketing team analyzes 6 months of clickstream:
> - Daily ETL via Glue: S3 raw logs → Parquet → COPY into Redshift fact table (`clicks`, ~5B rows).
> - Sort key `(date, user_id)`, distribution key `user_id`.
> - Dashboards via QuickSight run aggregations in seconds.

## Usage

### Connect (any Postgres client)

```bash
psql "host=mycluster.xxxx.ap-south-1.redshift.amazonaws.com port=5439 dbname=dev user=admin sslmode=require"
```

### Load data from S3 (COPY)

The canonical way to load data into Redshift:
```sql
COPY clicks
FROM 's3://bucket/clickstream/2026/05/'
IAM_ROLE 'arn:aws:iam::123456789012:role/RedshiftS3Role'
FORMAT AS PARQUET;
```

Bulk parallel load — far faster than INSERT.

### Spectrum: query S3 directly

```sql
CREATE EXTERNAL SCHEMA spectrum FROM DATA CATALOG
DATABASE 'analytics' IAM_ROLE 'arn:aws:iam::123456789012:role/RedshiftS3Role';

SELECT user_id, COUNT(*)
FROM spectrum.clicks  -- table defined in Glue Data Catalog
WHERE date BETWEEN '2026-01-01' AND '2026-05-01'
GROUP BY user_id;
```

No load step — pay $5 per TB scanned.

### Serverless

```bash
aws redshift-serverless create-workgroup \
  --workgroup-name analytics \
  --base-capacity 32 \
  --namespace-name analytics-ns
```

## Pricing

- **Provisioned RA3:** `ra3.xlplus` ≈ $3.26/hr per node + storage (~$0.024/GB-mo S3-backed).
- **Serverless:** $0.375 per RPU-hr. Min 8 RPUs.
- **Spectrum:** $5 per TB scanned.

Roughly $700-2,000/mo for a small cluster. Serverless lets you scale to zero between workloads.

## Performance tips

- Pick a **distribution key** that gives even data spread (e.g., `user_id` if writes are evenly distributed per user).
- Pick a **sort key** matching common `WHERE` predicates (often `date`).
- Use **Parquet** in S3 (columnar) — much faster than CSV for Spectrum.
- Use **`VACUUM`** + **`ANALYZE`** after big loads (or enable auto).
- **Materialized views** for repeated dashboard queries.
- **Result caching** is automatic — repeated queries return instantly.

## Gotchas

- **Not for OLTP.** Single-row inserts are slow; design batches.
- **Postgres-derived but not 1:1.** Some PG features missing, some Redshift-specific.
- **Long-running cluster costs even when idle.** Use Pause/Resume or Serverless.
- **Loading is best in chunks of 1 MB+ per slice** — small files are slow.
- **Network latency from outside the VPC** — use VPC endpoint.
- **Concurrency limits** — 50 concurrent queries default. Concurrency Scaling helps.

## Redshift vs Athena vs Aurora

| | Redshift | Athena | Aurora |
|---|---|---|---|
| Data shape | Curated, ETL'd | Files in S3 | Operational |
| Query speed | Fast (compiled, columnar) | Fast (Presto), pay-per-scan | OK for OLTP, slow for analytics |
| Cost model | Cluster $/hr or Serverless | Per TB scanned | Instance + storage |
| Best for | Recurring BI dashboards | Ad-hoc / data lake | App backend |

## Related

- [Athena](../10-analytics/athena.md)
- [Glue](../10-analytics/glue.md)
- [QuickSight](../10-analytics/quicksight.md)
- [Lake Formation](../10-analytics/lake-formation.md)
