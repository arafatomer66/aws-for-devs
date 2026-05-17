# AWS Batch

**TL;DR** — Run batch jobs (containerized) on EC2/Fargate at any scale. Queue, schedule, retry, and auto-scale compute. You write the jobs; AWS handles the bin-packing.

## What it is

A job-scheduling service for **batch computing**: stuff that runs to completion, not 24/7. Genomics pipelines, video transcoding, financial simulations, ML training, ETL.

## Why it exists

You *could* write your own scheduler on top of EC2 ASGs, but Batch already does it: job queues, priority, dependencies, retries, dynamic compute provisioning.

## Key concepts

- **Job** — one execution of a Docker container.
- **Job definition** — spec for a job (image, vCPU, memory, env, IAM role, retry).
- **Job queue** — jobs wait here, mapped to one or more compute environments.
- **Compute environment** — the EC2 / Fargate fleet that runs jobs.
  - **Managed** — Batch provisions/destroys EC2 for you.
  - **Unmanaged** — you bring your own fleet.
- **Array job** — fan out N copies (e.g., 1000 frames to render).
- **Job dependency** — "B starts only after A completes."

## Real-world example

> Bioinformatics team needs to align 5,000 DNA samples. Each alignment takes 30 mins on a 16-core machine. They:
>
> 1. Submit an **array job** of 5,000 children to the queue.
> 2. Batch spins up 50 c6i.4xlarge spot instances.
> 3. After 10 hours, all done. Batch shuts the fleet down.
> 4. Cost: a few hundred dollars instead of buying hardware.

## Usage

### Create a compute environment (Fargate)

```bash
aws batch create-compute-environment \
  --compute-environment-name fargate-spot \
  --type MANAGED --state ENABLED \
  --compute-resources type=FARGATE_SPOT,maxvCpus=256,subnets=subnet-a,subnet-b,securityGroupIds=sg-1234
```

### Create a queue

```bash
aws batch create-job-queue \
  --job-queue-name etl-queue --priority 1 --state ENABLED \
  --compute-environment-order order=1,computeEnvironment=fargate-spot
```

### Job definition

```json
{
  "jobDefinitionName": "transcode",
  "type": "container",
  "platformCapabilities": ["FARGATE"],
  "containerProperties": {
    "image": "123456789012.dkr.ecr.ap-south-1.amazonaws.com/transcoder:1.0",
    "resourceRequirements": [
      { "type": "VCPU",   "value": "1" },
      { "type": "MEMORY", "value": "2048" }
    ],
    "executionRoleArn": "arn:aws:iam::123456789012:role/ecsTaskExecutionRole",
    "jobRoleArn":      "arn:aws:iam::123456789012:role/transcode-job-role",
    "networkConfiguration": { "assignPublicIp": "DISABLED" }
  }
}
```

```bash
aws batch register-job-definition --cli-input-json file://transcode.json
```

### Submit jobs

```bash
aws batch submit-job \
  --job-name video-001 \
  --job-queue etl-queue \
  --job-definition transcode:1 \
  --container-overrides 'environment=[{name=INPUT,value=s3://bucket/in/001.mp4},{name=OUTPUT,value=s3://bucket/out/}]'

# Array job (1000 children)
aws batch submit-job \
  --job-name transcode-batch \
  --job-queue etl-queue \
  --job-definition transcode:1 \
  --array-properties size=1000
```

In your container, `AWS_BATCH_JOB_ARRAY_INDEX` env var tells the child which slice it is.

## Pricing

- **Batch itself is free.**
- You pay for the underlying compute: EC2 / Fargate / Spot.
- No control-plane overhead. The savings come from bin-packing + spot.

## When to use Batch vs alternatives

| Use case | Best fit |
|---|---|
| Long-running batch (> 15 min), array jobs | **Batch** |
| Short async tasks | Lambda (≤ 15 min) |
| Workflow orchestration (state machine) | Step Functions + Lambda/Batch |
| Pipelines with stages | AWS Glue (for data) / SageMaker Pipelines (for ML) |
| Kubernetes-native | Argo Workflows on EKS |

## Gotchas

- **Spot interruptions** — checkpoint and retry. Use `retryStrategy: { attempts: 5 }`.
- **Container exit code 0** = success. Anything else = failure → retry.
- **Logs go to CloudWatch** — group `/aws/batch/job`, one log stream per job.
- **EC2 launch type instances boot slow** (~2–4 min) — Fargate ~30s.
- **Resource requirements must match** allowed Fargate combos if using Fargate.

## Related

- [Step Functions](../06-messaging-integration/step-functions.md) — orchestrate Batch jobs
- [ECS](./ecs.md)
- [EMR](../10-analytics/emr.md) — different shape for Spark/Hive
- [SageMaker](../09-ml-ai/sagemaker.md) — ML-specific batch
