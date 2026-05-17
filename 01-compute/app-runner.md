# App Runner

**TL;DR** — Push a container or connect a GitHub repo, get a public HTTPS URL. Fully managed scaling, TLS, deploys. The simplest container hosting on AWS.

## What it is

A fully-managed container service. Two source modes:
- **Source code:** point at a GitHub repo with a `apprunner.yaml`. App Runner builds + deploys on every push.
- **Container image:** point at an ECR image. App Runner runs it.

It auto-creates an HTTPS endpoint, scales 0–25 instances based on requests, and handles all infra.

## Why it exists

ECS/EKS/Fargate are powerful but they require VPC + ALB + IAM + task defs + autoscaling config. App Runner gives you "git push → live URL" in 5 minutes.

## Key concepts

- **Service** — your running app.
- **Source** — GitHub repo or ECR image URI.
- **Auto-deployment** — auto-rebuild on git push.
- **Scaling configuration** — min/max instances, max concurrency per instance.
- **VPC connector** — to talk to private RDS / ElastiCache.
- **Custom domain** — bring your own domain, free ACM cert.
- **Observability configuration** — X-Ray tracing toggle.

## Real-world example

> A solo dev ships a Node API:
>
> 1. Pushes code to GitHub.
> 2. Creates App Runner service pointing at the repo.
> 3. Adds custom domain `api.myapp.dev`.
> 4. Connects to RDS Postgres via VPC connector.
>
> Total infra config in console: ~10 minutes. Cost: ~$5–25/mo depending on traffic.

## Usage

### From an image

```bash
aws apprunner create-service \
  --service-name my-api \
  --source-configuration '{
    "ImageRepository": {
      "ImageIdentifier": "123456789012.dkr.ecr.ap-south-1.amazonaws.com/my-api:latest",
      "ImageRepositoryType": "ECR",
      "ImageConfiguration": {
        "Port": "8080",
        "RuntimeEnvironmentVariables": { "NODE_ENV": "production" }
      }
    },
    "AutoDeploymentsEnabled": true
  }' \
  --instance-configuration '{ "Cpu": "1 vCPU", "Memory": "2 GB" }'
```

### From GitHub source

`apprunner.yaml` in your repo:
```yaml
version: 1.0
runtime: nodejs20
build:
  commands:
    build:
      - npm ci
      - npm run build
run:
  command: node dist/server.js
  network:
    port: 8080
  env:
    - name: NODE_ENV
      value: production
```

Then create via console: choose source = GitHub, branch = main, deployment = automatic.

## Pricing

- **Provisioned (always running):** ~$0.007/vCPU-hr + $0.0008/GB-hr (cheap idle rate).
- **Active (handling requests):** ~$0.064/vCPU-hr + $0.007/GB-hr.
- **Builds:** $0.005/build-min (only for source mode).

A 1 vCPU + 2 GB service with light traffic ~ **$10-25/mo**.

## Scaling

App Runner scales by concurrency, not CPU. Configure `Max Concurrency` per instance (e.g., 100 requests in flight = scale out).

Cold scale-from-zero: a few seconds for image, longer for first source build.

## When NOT to use App Runner

- Need WebSockets, gRPC streaming, or persistent connections beyond 120s.
- Need to mount EFS / persistent volumes.
- Need fine-grained network control (multiple ALBs, NLBs, etc.).
- Run >25 instances or really high traffic.

## Gotchas

- **No persistent storage.** Stateless only.
- **One container per service.** No sidecars.
- **`copilot` CLI** can build/deploy too — useful for multi-service projects.
- **VPC connector required** to talk to private RDS — adds a few seconds to cold start.
- **Limited regions** — verify availability before committing.

## Related

- [ECS / Fargate](./ecs.md) — when App Runner is too constrained
- [Elastic Beanstalk](./elastic-beanstalk.md) — older, non-container PaaS
- [Lambda](./lambda.md) — for true serverless functions
