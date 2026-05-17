# EMR — Elastic MapReduce

**TL;DR** — Managed Hadoop / Spark / Hive / Presto / HBase clusters. For big-data workloads where you want full control. EMR Serverless and EMR on EKS exist for less ops.

## What it is

A cluster service for big-data frameworks: Apache Spark, Hive, Presto/Trino, HBase, Flink, Hudi, Iceberg, Pig, Tez, Zeppelin notebooks. AWS provisions EC2 + installs the stack; you submit jobs.

## Modes

- **EMR on EC2 (Provisioned cluster)** — classic; you size masters + cores + tasks.
- **EMR Serverless** — submit jobs without sizing a cluster; AWS allocates.
- **EMR on EKS** — runs Spark on your EKS cluster.

## Key concepts

- **Master node** — coordinates.
- **Core nodes** — run tasks + store HDFS.
- **Task nodes** — extra compute, no HDFS (often Spot).
- **Step** — a job submitted to the cluster.
- **Bootstrap action** — install custom software on boot.
- **Instance fleets** — mix on-demand + spot, multiple instance types.

## Real-world example

> Big data team:
> - Daily 5-TB Spark job to compute user metrics.
> - EMR cluster: 1 master `m6g.xlarge`, 10 core `r6g.4xlarge`, 50 task `r6g.4xlarge` Spot.
> - Cluster spun up by Step Functions, runs the job, terminates.
> - Cost: ~$50 per run vs $500/mo for a permanent equivalent.

## Usage

```bash
aws emr create-cluster \
  --name daily-spark \
  --release-label emr-7.4.0 \
  --applications Name=Spark Name=Hive \
  --instance-fleets file://fleets.json \
  --use-default-roles \
  --auto-terminate \
  --steps Type=Spark,Args=["s3://my-jobs/main.py"] \
  --log-uri s3://my-emr-logs/
```

`fleets.json` defines instance types + Spot weights.

## EMR Serverless

```bash
aws emr-serverless create-application \
  --type SPARK \
  --release-label emr-7.4.0 \
  --name daily-spark

aws emr-serverless start-job-run \
  --application-id <id> \
  --execution-role-arn arn:aws:iam::..:role/EMRServerlessJobRole \
  --job-driver '{"sparkSubmit":{"entryPoint":"s3://my-jobs/main.py","sparkSubmitParameters":"--conf spark.executor.memory=4g"}}'
```

## Pricing

- **EMR on EC2:** EC2 + EBS prices + ~25% EMR markup.
- **EMR Serverless:** pay per vCPU-hr + memory-GB-hr while jobs run.
- **EMR on EKS:** pay for EKS + EC2 + EMR per-pod surcharge.

## EMR vs Glue

- **EMR** — full control, more knobs, supports more engines, faster for huge jobs.
- **Glue** — simpler, fully serverless, opinionated for ETL.

Many teams use Glue for daily ETL, EMR for occasional big jobs / data science.

## Gotchas

- **Cluster startup is slow** (5-10 min). Use Serverless or long-running cluster.
- **Spot capacity** can be tight for big instance types.
- **HDFS is ephemeral** — terminate cluster = data gone. Use S3 instead.
- **Bootstrap actions add startup time.**
- **Auto-terminate** is essential for transient clusters.

## Related

- [Glue](./glue.md)
- [Athena](./athena.md)
- [Lake Formation](./lake-formation.md)
