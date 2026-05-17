# MWAA — Managed Workflows for Apache Airflow

**TL;DR** — Managed Apache Airflow. Write DAGs in Python; AWS runs the scheduler, workers, and web server. For data pipeline orchestration. Step Functions is the AWS-native alternative.

## What it is

Airflow as a service. You upload your `dags/` folder to S3; MWAA picks up changes, runs DAGs on a schedule, exposes the Airflow UI. Supports Airflow 2.x with the standard ecosystem of providers (AWS, Snowflake, dbt, Spark, etc.).

## Key concepts

- **Environment** — one Airflow deployment. Pick size (Small/Medium/Large/XLarge).
- **DAG** — directed acyclic workflow (Python file).
- **Plugins / Requirements** — your custom Python deps via `requirements.txt` in S3.
- **Workers** — auto-scale based on queued tasks.
- **Networking** — runs in your VPC.

## When MWAA vs Step Functions vs Glue Workflows

- **MWAA** — your team already speaks Airflow; you want the Airflow ecosystem (operators, sensors, hooks).
- **Step Functions** — you want AWS-native, cheaper, simpler, visual; ok writing in ASL or CDK.
- **Glue Workflows** — strictly ETL within Glue. Limited but free.

For new AWS-only teams: **Step Functions usually wins** on cost and simplicity. MWAA is for Airflow-heavy data orgs.

## Real-world example

> Data team:
> - Daily ETL DAG: extract from MySQL → Glue → Snowflake.
> - Hourly DAG: aggregate metrics → Redshift.
> - Used Airflow on EC2 self-hosted; switched to MWAA to stop patching.

## Usage

1. Upload DAGs to S3 (`s3://my-mwaa/dags/`).
2. Create environment:

```bash
aws mwaa create-environment --name etl --airflow-version 2.10.1 \
  --execution-role-arn arn:aws:iam::..:role/MwaaExecRole \
  --source-bucket-arn arn:aws:s3:::my-mwaa \
  --dag-s3-path dags --network-configuration ... \
  --environment-class mw1.small
```

3. UI URL is returned; sign in via IAM/SSO.

Sample DAG:
```python
from airflow import DAG
from airflow.providers.amazon.aws.operators.glue import GlueJobOperator
from datetime import datetime

with DAG("daily_etl", start_date=datetime(2026,1,1), schedule="0 2 * * *", catchup=False) as dag:
    run_etl = GlueJobOperator(task_id="glue_etl", job_name="etl-orders", aws_conn_id="aws_default")
```

## Pricing

- **`mw1.small`** ≈ $0.49/hr ≈ $355/mo for the env, plus worker autoscale costs.
- Significantly pricier than Step Functions for similar volumes.

## Gotchas

- **VPC-private by default.** Plan endpoints for S3, ECR, etc.
- **`requirements.txt` install** runs on every scheduler/worker — slow startup with heavy deps.
- **Airflow version upgrades** are blue/green env swaps.
- **Worker autoscale is pool-based** — under heavy bursts tasks queue.

## Related

- [Step Functions](../06-messaging-integration/step-functions.md)
- [Glue](./glue.md)
- [EMR](./emr.md)
