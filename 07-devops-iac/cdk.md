# CDK — Cloud Development Kit

**TL;DR** — Define AWS infra in TypeScript / Python / Java / Go / C#. Synthesizes to CloudFormation. Real code → loops, conditionals, reusable constructs, IDE autocompletion.

## What it is

A framework for IaC in programming languages. You instantiate **Constructs** (classes representing AWS resources or groups); `cdk synth` produces a CFN template; `cdk deploy` runs it.

## Why it exists

CFN/Terraform DSLs are verbose, lossy on conditionals/loops, and hard to abstract. CDK uses TypeScript (or others) — first-class loops, types, IDE help, sharable constructs, tests.

## Key concepts

- **Construct** — a building block. Three levels:
  - **L1** — `CfnXxx` — 1:1 with CFN resource (e.g., `CfnBucket`).
  - **L2** — `Xxx` — higher-level (e.g., `Bucket` with sensible defaults).
  - **L3 / Patterns** — opinionated bundles (e.g., `ApplicationLoadBalancedFargateService`).
- **Stack** — a CFN stack, but defined in code.
- **App** — collection of stacks.
- **Construct tree** — composition hierarchy.
- **Aspects** — visitors that mutate/check resources (e.g., "tag everything in this stack").
- **Bootstrap** — one-time per region/account; creates a CDK toolkit stack (S3 bucket for assets, ECR repo, roles).

## Real-world example

> ShareDeal's CDK stack:
>
> ```ts
> const vpc = new ec2.Vpc(this, "Vpc", { maxAzs: 3, natGateways: 1 });
>
> const cluster = new ecs.Cluster(this, "Cluster", { vpc });
>
> const svc = new ecs_patterns.ApplicationLoadBalancedFargateService(this, "Api", {
>   cluster, memoryLimitMiB: 1024, cpu: 512, desiredCount: 4,
>   taskImageOptions: { image: ecs.ContainerImage.fromAsset("./service") },
>   domainName: "api.selefe.com", domainZone: hostedZone, certificate,
> });
>
> const db = new rds.DatabaseCluster(this, "Db", {
>   engine: rds.DatabaseClusterEngine.auroraPostgres({ version: rds.AuroraPostgresEngineVersion.VER_16_4 }),
>   credentials: rds.Credentials.fromGeneratedSecret("sdadmin"),
>   writer: rds.ClusterInstance.serverlessV2("Writer"),
>   vpc,
> });
> db.connections.allowDefaultPortFrom(svc.service);
> ```
>
> ~20 lines of code → 50+ CFN resources synthesized correctly.

## Usage

### Install + init

```bash
npm install -g aws-cdk

mkdir my-stack && cd my-stack
cdk init app --language typescript
```

### Bootstrap (once per account/region)

```bash
cdk bootstrap aws://123456789012/ap-south-1
```

### Common commands

```bash
cdk ls                       # list stacks
cdk synth                    # emit CFN
cdk diff                     # diff against deployed
cdk deploy                   # deploy current
cdk deploy --all             # all stacks
cdk destroy                  # tear down
cdk watch                    # hot reload (for Lambdas)
```

### Structure

```
my-stack/
├── bin/my-stack.ts          # entry point (App + Stacks)
├── lib/
│   ├── network-stack.ts
│   ├── data-stack.ts
│   └── app-stack.ts
├── cdk.json
└── package.json
```

### Cross-stack references

```ts
// data-stack.ts
this.bucket = new s3.Bucket(this, "Bucket");

// app-stack.ts
new lambda.Function(this, "Fn", { environment: { BUCKET: dataStack.bucket.bucketName } });
dataStack.bucket.grantRead(this.fn);  // CDK manages the IAM
```

CDK handles the CFN cross-stack exports automatically.

## CDK Pipelines (CI/CD for CDK)

```ts
const pipeline = new pipelines.CodePipeline(this, "Pipeline", {
  synth: new pipelines.ShellStep("Synth", {
    input: pipelines.CodePipelineSource.gitHub("arafatomer66/sd-infra", "main"),
    commands: ["npm ci", "npx cdk synth"],
  }),
});
pipeline.addStage(new MyStage(this, "Dev"));
pipeline.addStage(new MyStage(this, "Prod"), { pre: [new pipelines.ManualApprovalStep("Approve")] });
```

## Testing

```ts
import { Template } from "aws-cdk-lib/assertions";

const app = new App();
const stack = new MyStack(app, "TestStack");
const template = Template.fromStack(stack);

template.hasResourceProperties("AWS::S3::Bucket", { VersioningConfiguration: { Status: "Enabled" } });
template.resourceCountIs("AWS::Lambda::Function", 3);
```

## Pricing

- **CDK is free.** You pay for resources.

## Gotchas

- **`cdk synth` failures often = TypeScript issues.** Make sure your code compiles.
- **Bootstrap stack** — don't delete it; CDK won't work.
- **Stack drift** isn't auto-detected. Use CFN drift detection.
- **Construct IDs are identity.** Renaming a construct ID = AWS deletes the old resource and creates a new one. Be careful.
- **Asset bundling** — Lambda code is built/zipped on `cdk deploy` (uses Docker for some runtimes). Slow on first run; cached after.
- **Cross-stack references** create CFN exports; circular references are fatal — refactor.
- **Don't put secrets in CDK code.** Use Secrets Manager or Parameter Store; reference at deploy.

## Related

- [CloudFormation](./cloudformation.md)
- [SAM](./sam.md) — serverless-flavored CDK alternative
- [CodePipeline](./codepipeline.md)
