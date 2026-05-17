# Fargate — Serverless Containers

**TL;DR** — Run containers on ECS or EKS without managing EC2 instances. AWS provisions the compute, you specify CPU/RAM. Pay per task-second.

## What it is

Fargate is a **launch type / compute engine** for ECS and EKS. Instead of attaching EC2 worker nodes, you say "run this container with 1 vCPU and 2 GB RAM" and AWS does it. No SSH, no patching, no instance sizing.

## Why it exists

EC2 is great but you have to:
- Right-size instances.
- Patch the OS.
- Manage Auto Scaling Groups.
- Worry about bin-packing (fitting tasks onto instances).

Fargate abstracts all that away — just request CPU/RAM per task.

## Key concepts

Same as ECS/EKS, but instead of choosing EC2 launch type, you choose **FARGATE** in the task definition:
- `requiresCompatibilities: ["FARGATE"]`
- `networkMode: awsvpc` (mandatory)
- `cpu` and `memory` in specific allowed combos (256 CPU + 512 MiB, 512+1024, 1024+2048, etc.)

## Fargate vs Fargate Spot

- **Fargate** — on-demand, full price.
- **Fargate Spot** — ~70% cheaper, can be reclaimed with 2-min notice. Great for batch / stateless workers.

## Real-world example

> ShareDeal's image-processing service:
>
> - Bursty load: most days a few hundred images, but during campaigns thousands per minute.
> - Used to be EC2 — paying for idle servers most of the time.
> - Moved to ECS Fargate Spot: tasks spin up only when SQS queue depth grows. Costs dropped 80%.

## Usage

Fargate is configured via the **task definition**, not a separate API. See [ECS](./ecs.md) for the full example. Key bits:

```json
{
  "requiresCompatibilities": ["FARGATE"],
  "networkMode": "awsvpc",
  "cpu": "1024",
  "memory": "2048",
  "runtimePlatform": {
    "cpuArchitecture": "ARM64",  // Graviton, ~20% cheaper
    "operatingSystemFamily": "LINUX"
  }
}
```

### Run as one-off task (no service)

```bash
aws ecs run-task \
  --cluster prod \
  --task-definition migration:3 \
  --launch-type FARGATE \
  --network-configuration "awsvpcConfiguration={subnets=[subnet-a],securityGroups=[sg-1234],assignPublicIp=ENABLED}"
```

Use this for database migrations, one-off batch jobs, etc.

### Run as long-running service

Wrap it in an ECS service (see [ECS](./ecs.md#deploy)) with `launchType: FARGATE` or use a capacity provider strategy:

```bash
--capacity-provider-strategy "capacityProvider=FARGATE,weight=1,base=2" "capacityProvider=FARGATE_SPOT,weight=4"
```

This keeps 2 on-demand baseline tasks, then 1:4 ratio of regular:Spot for scale-out.

## Allowed CPU + memory combos

| CPU | Memory options (MiB) |
|---|---|
| 256 (0.25 vCPU) | 512, 1024, 2048 |
| 512 (0.5 vCPU)  | 1024 – 4096 (1 GiB increments) |
| 1024 (1 vCPU)   | 2048 – 8192 (1 GiB increments) |
| 2048 (2 vCPU)   | 4096 – 16384 (1 GiB increments) |
| 4096 (4 vCPU)   | 8192 – 30720 (1 GiB increments) |
| 8192 (8 vCPU)   | 16384 – 61440 (4 GiB increments) — Linux only |
| 16384 (16 vCPU) | 32768 – 122880 (8 GiB increments) — Linux only |

## Pricing

- **CPU:** ~$0.04048/vCPU-hr (Linux x86, us-east-1).
- **Memory:** ~$0.004445/GB-hr.
- **Graviton (ARM):** ~20% cheaper.
- **Spot:** ~70% off.
- **Storage:** 20 GiB ephemeral free per task; up to 200 GiB extra at $0.000111/GiB-hr.

Example: 1 vCPU + 2 GB task running 24/7 ≈ $30/mo. 0.25 vCPU + 0.5 GB ≈ $9/mo.

## Gotchas

- **No GPU on Fargate (yet).** Use EC2 launch type for GPU workloads.
- **No privileged containers.** Can't run Docker-in-Docker.
- **No host volumes** — use EFS for persistent shared storage.
- **First-task cold start** ~45–90 seconds. Subsequent tasks much faster.
- **Allowed CPU/RAM combos are discrete** — you can't ask for "750 MB RAM."
- **Egress costs** — Fargate tasks in private subnets need a NAT for ECR pulls (or use VPC endpoints).
- **ENI per task** — Fargate uses awsvpc mode, each task gets a VPC IP. Plan subnet sizes.
- **Logs to CloudWatch by default** — add `awslogs-create-group: true` or pre-create the log group.

## Related

- [ECS](./ecs.md)
- [EKS](./eks.md)
- [App Runner](./app-runner.md) — even simpler if you just need a web app
