# CodeDeploy

**TL;DR** — Automated deployments to EC2, Lambda, ECS, or on-prem. Blue/green, canary, linear strategies with auto-rollback.

## What it is

A deployment orchestrator. Given a deployment target + an artifact (revision), it rolls out using your chosen strategy. Handles draining, health checks, rollback.

## Three compute platforms

### EC2 / On-Prem
- Agent runs on each instance.
- `appspec.yml` describes lifecycle hooks (`BeforeInstall`, `AfterInstall`, `ApplicationStart`, `ValidateService`).
- Deployments: in-place or blue/green.

### Lambda
- Deploys new version by shifting traffic via an alias (canary, linear, all-at-once).
- Pre/post-traffic hooks (Lambdas that validate).

### ECS
- Blue/green: new task set + ALB target group + traffic shift + drain old set.
- Pre/post-traffic Lambda hooks.

## Strategies

- **All-at-once** — fast, no canary, riskier.
- **Linear 10% every X min** — incremental.
- **Canary 10% then 90%** — small slice for X min, then rest.
- **Blue/green** — full new set, switch over, keep old for rollback.

## Real-world example

> ECS service deploys via CodePipeline → CodeDeploy:
> - Build pipeline pushes new image to ECR.
> - CodeDeploy creates a new green task set.
> - ALB sends 10% traffic to green for 5 min.
> - CloudWatch alarm watches 5xx rate; if it spikes → rollback.
> - If clean, shift 100% to green; old set drained after 10 min.

## Usage (ECS blue/green)

`appspec.yaml`:
```yaml
version: 0.0
Resources:
  - TargetService:
      Type: AWS::ECS::Service
      Properties:
        TaskDefinition: <TASK_DEFINITION>
        LoadBalancerInfo:
          ContainerName: api
          ContainerPort: 8080
Hooks:
  - BeforeAllowTraffic: arn:aws:lambda:..:function:pre-traffic-hook
  - AfterAllowTraffic:  arn:aws:lambda:..:function:post-traffic-hook
```

## Pricing

- **Free** for EC2/On-Prem (you pay for instances).
- **Free** for Lambda + ECS deployments.

## Gotchas

- **Lambda traffic shifting needs an alias** — your app has to invoke the alias, not `$LATEST`.
- **ECS blue/green requires ALB or NLB target groups** — two of them.
- **Rollback isn't instant** — depends on health check intervals.
- **In-place EC2 deploys** stop the service briefly per instance. Use blue/green for zero-downtime.

## Related

- [CodePipeline](./codepipeline.md)
- [CodeBuild](./codebuild.md)
- [ECS](../01-compute/ecs.md)
- [Lambda](../01-compute/lambda.md)
