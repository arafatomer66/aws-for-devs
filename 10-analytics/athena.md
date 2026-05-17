# Athena

**TL;DR** — Run SQL on S3 data. Serverless, pay per TB scanned. Presto/Trino under the hood. No clusters to manage.

## What it is

A query engine that reads data directly from S3 using SQL (Presto/Trino dialect). Define a table schema in the **Glue Data Catalog**; Athena reads the underlying files (CSV, JSON, Parquet, ORC, Avro).

## Why it exists

If you don't need a constantly-running warehouse, Athena gives you ad-hoc SQL on S3 with zero infra. Perfect for data lakes, log analysis, CloudTrail forensics, occasional analytics.

## Key concepts

- **Workgroup** — settings + result location + per-query cost limits.
- **Data Catalog** — schema metadata (Glue Data Catalog is default).
- **External table** — points at S3 location with a schema.
- **Partitions** — folders by date / region / etc. Athena prunes them based on `WHERE`.
- **Federated queries** — query non-S3 sources (RDS, DynamoDB, OpenSearch) via Lambda connectors.
- **Athena for Spark** — run Spark notebooks too (separate engine version).

## Real-world example

> ShareDeal logs ALB access logs to S3.
> - Glue crawler infers schema.
> - Athena query:
>   ```sql
>   SELECT count(*) AS five_hundreds, client_ip
>   FROM alb_logs
>   WHERE elb_status_code = 500
>     AND date BETWEEN '2026-05-17' AND '2026-05-18'
>   GROUP BY client_ip ORDER BY 1 DESC LIMIT 20;
>   ```
> - Cost: $0.005 because data is partitioned by `date`, scanning ~1 GB.

## Usage

### Create table

```sql
CREATE EXTERNAL TABLE alb_logs (
  type string,
  time string,
  elb string,
  client_ip string,
  client_port int,
  target_ip string,
  ...
)
PARTITIONED BY (date string)
ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.RegexSerDe'
WITH SERDEPROPERTIES ( "input.regex" = "..." )
LOCATION 's3://my-bucket/alb-logs/';

MSCK REPAIR TABLE alb_logs;   -- discover partitions
```

Or let a Glue Crawler infer it.

### Query

```sql
SELECT * FROM alb_logs WHERE date='2026-05-18' LIMIT 10;
```

### Save results

Results auto-save to an S3 location (configurable per workgroup).

### Athena SDK call (Node)

```js
import { AthenaClient, StartQueryExecutionCommand, GetQueryResultsCommand } from "@aws-sdk/client-athena";
const ath = new AthenaClient({ region: "ap-south-1" });
const start = await ath.send(new StartQueryExecutionCommand({
  QueryString: "SELECT count(*) FROM alb_logs WHERE date='2026-05-18'",
  WorkGroup: "primary",
  ResultConfiguration: { OutputLocation: "s3://my-athena-results/" },
}));
// Poll GetQueryExecution; then GetQueryResults
```

## Pricing

- **$5 per TB scanned.**
- **Spark engine:** per-DPU-hr (similar to Glue).
- **No charge for DDL** (CREATE TABLE, etc.).

## Cost optimization

The way to control Athena costs is to **scan less data**:
1. **Partition** by frequently-filtered columns (date, region).
2. Use **Parquet** or **ORC** — columnar, only reads needed columns.
3. **Compress** files (Snappy, Gzip).
4. **Filter early** with `WHERE`.
5. **Project specific columns** (avoid `SELECT *`).
6. Use **partition projection** to skip metadata calls for high-partition tables.

A million-row CSV scanned = ~$$. The same data as partitioned Parquet = pennies.

## Athena vs Redshift vs RDS

| | Athena | Redshift | RDS |
|---|---|---|---|
| Data shape | Files in S3 | Curated/ETL'd | Operational |
| Concurrency | Limited | High | High |
| Latency | Seconds-minutes | Sub-second to seconds | Sub-second |
| Cost model | Per TB scanned | Cluster $/hr or Serverless | Instance $/hr |
| Best for | Ad-hoc / data lake | Recurring BI | App backend |

## Gotchas

- **`SELECT *` scans everything.** Project columns explicitly.
- **Small files = many round-trips.** Compact small files (Glue Job, EMR) into 100 MB+ chunks.
- **Partition projection** beats `MSCK REPAIR TABLE` for huge partition counts.
- **Federated queries** invoke Lambdas — additional cost + latency.
- **Per-query DML write limits** — for write-heavy workloads use INSERT to Iceberg tables, not raw S3 putters.
- **Iceberg / Hudi support** is available — schema evolution + ACID on S3.

## Related

- [Glue](./glue.md)
- [Redshift](../03-database/redshift.md)
- [Lake Formation](./lake-formation.md)
- [QuickSight](./quicksight.md)
