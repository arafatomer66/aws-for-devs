# AWS Glue

**TL;DR** — Serverless ETL + Data Catalog. Crawl S3 / RDS / etc. to populate the catalog, then run Spark/Python ETL jobs without managing clusters.

## What it is

A bundle:
- **Data Catalog** — Hive-compatible metastore used by Athena, EMR, Redshift Spectrum, etc.
- **Crawlers** — auto-infer schemas in S3/RDS and register tables.
- **Jobs** — Spark or Python shell ETL.
- **Studio** — visual ETL builder.
- **DataBrew** — no-code data prep.
- **Schema Registry** — for Kafka / Kinesis schemas.

## Key concepts

- **Database / Table** in Data Catalog.
- **Crawler** — scheduled scan of an S3 prefix or DB.
- **Job** — Spark (PySpark or Scala) or Python shell.
- **Worker types:** `G.1X`, `G.2X`, `G.4X`, `G.8X` (Spark workers).
- **DPU** — Data Processing Unit (4 vCPU + 16 GB).
- **Job bookmarks** — track processed data so reruns are incremental.
- **Glue triggers** — schedule or event-based.
- **Glue Workflows** — orchestrate multiple jobs + crawlers.

## Real-world example

> Daily ETL pipeline:
> - Crawler scans `s3://raw/orders/2026/05/18/` → infers Parquet schema → registers in catalog.
> - Glue PySpark job reads raw orders, joins with product table from RDS, writes curated Parquet to `s3://curated/orders/`.
> - Athena and QuickSight query the curated table.

## Usage

### A Glue PySpark job

```python
from awsglue.context import GlueContext
from pyspark.context import SparkContext

gc = GlueContext(SparkContext.getOrCreate())

orders = gc.create_dynamic_frame.from_catalog(database="raw", table_name="orders")
products = gc.create_dynamic_frame.from_catalog(database="raw", table_name="products")

joined = orders.join(["product_id"], ["id"], products)

gc.write_dynamic_frame.from_options(
  frame=joined,
  connection_type="s3",
  connection_options={"path": "s3://curated/orders/", "partitionKeys": ["date"]},
  format="parquet",
)
```

Save as `etl.py`, upload to S3, create job pointing at it.

### Trigger

```bash
aws glue create-trigger --name daily-etl --type SCHEDULED --schedule "cron(0 2 * * ? *)" \
  --actions JobName=etl-orders --start-on-creation
```

## Pricing

- **Spark jobs:** $0.44 per DPU-hour, billed per second (min 1 min for v2.0+, 10 min for older).
- **Python shell jobs:** $0.44 per DPU-hr, 0.0625 or 1 DPU.
- **Crawlers:** same DPU rate.
- **Data Catalog:** first 1M objects + 1M requests/mo free; cheap thereafter.

## Glue vs EMR vs Lambda vs Step Functions

- **Glue** — serverless Spark, opinionated for ETL. Best for batch.
- **EMR** — clusters with full Hadoop/Spark control, faster for big jobs at scale.
- **Lambda** — tiny ETL functions (under 15 min).
- **Step Functions** — orchestrate any of the above.

## Gotchas

- **Cold start** — Spark job startup is ~30 s.
- **Bookmarks** help avoid reprocessing but require careful key design.
- **Dev endpoints / Notebooks** are pricey — stop them.
- **`G.025X` workers** (Glue 3+) are cheap for small Python shell jobs.
- **Crawlers can create messy catalogs** — they sample data; manual schemas often cleaner.

## Related

- [Athena](./athena.md)
- [EMR](./emr.md)
- [Lake Formation](./lake-formation.md)
- [Redshift](../03-database/redshift.md)
