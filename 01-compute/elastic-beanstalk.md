# Elastic Beanstalk

**TL;DR** — Heroku-on-AWS. Upload your code, it provisions EC2 + ELB + ASG + RDS for you. Older but still useful for "I just want my Java/Node/Python app online."

## What it is

A PaaS. You give it a zip / Docker image and a platform (Node 20, Python 3.12, Java 21, Go, Ruby, PHP, Docker, .NET on Linux/Windows). Beanstalk creates:
- EC2 instances behind an ALB.
- An Auto Scaling Group.
- Optional RDS instance.
- CloudWatch metrics + logs.
- IAM roles + security groups.

You can still SSH in, change `.ebextensions/`, override anything. Less magic than App Runner, more control.

## Why it exists

Pre-Lambda, pre-Fargate, this was AWS's answer to Heroku. Still solid for traditional web apps that aren't ready for serverless.

## Key concepts

- **Application** — top-level container.
- **Environment** — a deployment (e.g. `prod`, `staging`). Each has its own ELB and ASG.
- **Version** — a specific code zip.
- **Platform** — managed runtime (e.g., "Node.js 20 running on Amazon Linux 2023").
- **Configuration** — env vars, instance type, scaling triggers (`.ebextensions/`).
- **Saved configuration** — reusable env templates.
- **Worker tier** — env type that processes SQS messages instead of HTTP.

## Real-world example

> A 5-engineer Java/Spring Boot team. They:
>
> - Push a JAR to Beanstalk.
> - Beanstalk runs it on 3× t3.medium behind an ALB.
> - RDS PostgreSQL attached as the data tier.
> - Blue/green deploys via `eb deploy --staged`.
>
> No K8s, no Fargate config, no CDK. Works.

## Usage

### Install + init

```bash
pip install awsebcli
eb init -p "Node.js 20" my-app --region ap-south-1
```

### Create environment

```bash
eb create my-app-prod \
  --instance-type t3.small \
  --min-instances 2 --max-instances 6 \
  --elb-type application
```

### Deploy

```bash
git add . && git commit -m "feat"
eb deploy
```

### Get URL

```bash
eb status
# CNAME: my-app-prod.eba-xxxx.ap-south-1.elasticbeanstalk.com
```

### Custom hooks (`.ebextensions/01-setup.config`)

```yaml
option_settings:
  aws:autoscaling:asg:
    MinSize: 2
    MaxSize: 6
  aws:autoscaling:launchconfiguration:
    InstanceType: t3.medium
  aws:elasticbeanstalk:application:environment:
    NODE_ENV: production
container_commands:
  01_migrate:
    command: "npx prisma migrate deploy"
    leader_only: true
```

## Pricing

- **Beanstalk itself is free.**
- You pay for EC2, ELB, S3, RDS, etc. that it creates.

## Beanstalk vs alternatives

| | Beanstalk | App Runner | Fargate | EKS |
|---|---|---|---|---|
| Setup time | Minutes | Minutes | Hour | Days |
| Control | High | Low | High | Total |
| Best for | Traditional web apps | Container web apps | Microservices | k8s shops |

## Gotchas

- **Slower deploys** than containers (rebuilds a whole environment).
- **Hard to mix in custom infrastructure** alongside (better: CDK).
- **The default RDS attached to an env can be deleted with the env.** Detach it for prod.
- **Platform updates** — managed updates can restart your fleet. Configure window.
- **Less actively developed** by AWS — clearly being de-emphasized in favor of App Runner / ECS.

## Related

- [App Runner](./app-runner.md) — newer, container-focused
- [ECS](./ecs.md) — when you outgrow Beanstalk
- [Lightsail](./lightsail.md) — when Beanstalk feels heavy
