# ECS — Elastic Container Service

**TL;DR** — AWS's own container orchestrator. Run Docker containers without managing your own Kubernetes. Simpler than EKS, more opinionated.

## What it is

ECS is a managed control plane that schedules Docker containers across a fleet. The fleet can be:
- **EC2 launch type** — you manage the underlying EC2 instances.
- **Fargate launch type** — AWS manages the underlying compute (serverless).

## Why it exists

Kubernetes is powerful but complex. ECS gives you ~80% of K8s value for ~20% of the operational pain. AWS-native, integrates tightly with IAM, ALB, CloudWatch, etc.

## Key concepts

- **Task definition** — a JSON spec like a Pod (container image, CPU, RAM, env vars, IAM role).
- **Task** — a running instance of a task definition.
- **Service** — keeps N copies of a task running, integrates with load balancers, handles rolling deploys.
- **Cluster** — logical group of tasks/services + the EC2 capacity or Fargate.
- **Capacity provider** — strategy for placing tasks (EC2, Fargate, Fargate Spot).
- **Service Connect / Service Discovery** — service-to-service discovery (via DNS).
- **Task role** — IAM role your container code uses (not the agent role).
- **Execution role** — IAM role ECS uses to pull images / write logs.

## ECS vs EKS

| | ECS | EKS |
|---|---|---|
| API | AWS-proprietary | Kubernetes |
| Learning curve | Low | High |
| Portability | AWS-only | Run anywhere k8s runs |
| Add-ons | Fewer, AWS-curated | Huge ecosystem (Helm, operators) |
| Cost | No control-plane fee | $0.10/hr per cluster |
| Best for | "Just run my containers" | "We're a k8s shop" |

## Real-world example

> ShareDeal runs its API as an ECS service on Fargate:
>
> - Task definition: Node.js image, 1 vCPU, 2 GB RAM, env vars from Secrets Manager.
> - Service: desired-count = 4, behind an ALB, 3 AZs.
> - Auto-scaling: scale to 8 if CPU > 60% for 2 min.
> - Deploys via CodePipeline → ECS rolling update.

## Usage

### Define a task

`task-definition.json`:
```json
{
  "family": "api",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "512",
  "memory": "1024",
  "executionRoleArn": "arn:aws:iam::123456789012:role/ecsTaskExecutionRole",
  "taskRoleArn":      "arn:aws:iam::123456789012:role/api-task-role",
  "containerDefinitions": [{
    "name": "api",
    "image": "123456789012.dkr.ecr.ap-south-1.amazonaws.com/api:latest",
    "essential": true,
    "portMappings": [{ "containerPort": 8080 }],
    "environment": [{ "name": "NODE_ENV", "value": "production" }],
    "secrets": [{ "name": "DB_PASS", "valueFrom": "arn:aws:secretsmanager:..:secret:db-pass" }],
    "logConfiguration": {
      "logDriver": "awslogs",
      "options": {
        "awslogs-group": "/ecs/api",
        "awslogs-region": "ap-south-1",
        "awslogs-stream-prefix": "api"
      }
    }
  }]
}
```

### Deploy

```bash
aws ecs register-task-definition --cli-input-json file://task-definition.json

aws ecs create-service \
  --cluster prod \
  --service-name api \
  --task-definition api:1 \
  --desired-count 4 \
  --launch-type FARGATE \
  --network-configuration "awsvpcConfiguration={subnets=[subnet-a,subnet-b,subnet-c],securityGroups=[sg-1234],assignPublicIp=DISABLED}" \
  --load-balancers "targetGroupArn=arn:...,containerName=api,containerPort=8080"

# Roll out a new version
aws ecs update-service --cluster prod --service api --task-definition api:2
```

### CDK

```ts
const cluster = new ecs.Cluster(this, 'Cluster', { vpc });

new ecs_patterns.ApplicationLoadBalancedFargateService(this, 'Api', {
  cluster,
  cpu: 512,
  memoryLimitMiB: 1024,
  desiredCount: 4,
  taskImageOptions: {
    image: ecs.ContainerImage.fromEcrRepository(repo),
    containerPort: 8080,
  },
});
```

The `ApplicationLoadBalancedFargateService` pattern is the easiest way to ship a containerized web app.

## Service auto-scaling

```bash
aws application-autoscaling register-scalable-target \
  --service-namespace ecs \
  --resource-id service/prod/api \
  --scalable-dimension ecs:service:DesiredCount \
  --min-capacity 2 --max-capacity 20

aws application-autoscaling put-scaling-policy \
  --policy-name cpu-target \
  --service-namespace ecs \
  --resource-id service/prod/api \
  --scalable-dimension ecs:service:DesiredCount \
  --policy-type TargetTrackingScaling \
  --target-tracking-scaling-policy-configuration '{
    "TargetValue": 60.0,
    "PredefinedMetricSpecification": { "PredefinedMetricType": "ECSServiceAverageCPUUtilization" }
  }'
```

## Pricing

- **EC2 launch type:** pay for EC2 + EBS. ECS control plane is free.
- **Fargate launch type:** ~$0.04/vCPU-hr + $0.004/GB-hr. Pay only while tasks run.
- **Fargate Spot:** ~70% off, can be interrupted.

## Gotchas

- **awsvpc network mode = one ENI per task.** Tasks get IPs from your VPC subnets. ENI quota can bite you.
- **Pulling private images** requires `executionRoleArn` with ECR pull perms.
- **Health checks must agree:** container health check + target group health check + service definition.
- **Rolling deploys can stall** if min-healthy-percent + max-percent are misconfigured. Set 100/200 for safety.
- **CPU/memory units** — 1024 CPU = 1 vCPU. Memory in MiB.
- **Logs go to CloudWatch by default** — they cost $0.50/GB ingested. Use Firelens to ship to S3/OpenSearch for cheap.

## Related

- [Fargate](./fargate.md) — the serverless launch type
- [EKS](./eks.md) — Kubernetes alternative
- [ECR](#) — Elastic Container Registry (Docker image registry)
- [ALB](../04-networking/elb.md) — fronting ECS services
- [App Runner](./app-runner.md) — even simpler container hosting
