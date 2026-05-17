# Timestream

**TL;DR** — Managed time-series database. For IoT, telemetry, financial ticks, app metrics. Auto-tiers from memory → magnetic storage. Query with SQL.

Two flavors as of 2024:
- **Timestream for LiveAnalytics** — original, serverless.
- **Timestream for InfluxDB** — managed InfluxDB Enterprise (acquired-ish in 2024).

## Why it exists

Time-series workloads (millions of points/sec, range queries by time, downsampling, retention windows) are awkward in relational DBs. Specialized time-series DBs handle this natively.

## Key concepts (LiveAnalytics)

- **Database** → **Table** → **Records**.
- Each record has: dimensions (tags like `device=abc`), measure (value), time.
- **Storage tiers:** memory store (recent, fast) → magnetic store (cold).
- **Retention** — set per tier; data ages out automatically.
- **Scheduled queries** — pre-aggregate into rollup tables.

## Real-world example

> IoT fleet of 100k sensors emitting temperature every 10s:
> - Ingest into Timestream from Kinesis or IoT Core.
> - Memory tier: last 24 h for live dashboards.
> - Magnetic tier: last 2 years for trending.
> - Scheduled query rolls up to hourly averages.

## Usage

```bash
aws timestream-write create-database --database-name fleet
aws timestream-write create-table --database-name fleet --table-name temperature \
  --retention-properties '{"MemoryStoreRetentionPeriodInHours":24,"MagneticStoreRetentionPeriodInDays":730}'
```

Write (SDK):
```python
import boto3, time
client = boto3.client("timestream-write", region_name="ap-south-1")
client.write_records(
  DatabaseName="fleet", TableName="temperature",
  Records=[{
    "Dimensions": [{"Name":"device","Value":"abc"},{"Name":"region","Value":"ap-south-1"}],
    "MeasureName":"temp_c","MeasureValue":"22.5","MeasureValueType":"DOUBLE",
    "Time": str(int(time.time()*1000)), "TimeUnit":"MILLISECONDS"
  }]
)
```

Query:
```sql
SELECT BIN(time, 1h) AS bucket, AVG(measure_value::double) AS avg_temp
FROM "fleet"."temperature"
WHERE device = 'abc' AND time > ago(7d)
GROUP BY BIN(time, 1h)
ORDER BY bucket;
```

## Pricing

- **Writes:** $0.50 per million records (1 KB).
- **Memory store:** $0.036/GB-hr (~$26/GB-mo).
- **Magnetic store:** $0.03/GB-mo.
- **Queries:** $0.01/GB scanned.

## Gotchas

- **Append-only semantics.** You can't UPDATE a record (must write a new version).
- **Memory tier is pricey** — keep retention short for hot data only.
- **SDK supports batch writes** — always batch, don't single-write per request.

## Related

- [IoT Core](#)
- [Kinesis](../06-messaging-integration/kinesis.md)
- [CloudWatch](../08-monitoring-observability/cloudwatch.md) — for your *own* metrics, simpler
