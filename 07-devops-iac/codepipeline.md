# CodePipeline

**TL;DR** — CI/CD orchestrator. Stages with actions: Source → Build → Deploy → Approve. Glues CodeBuild + CodeDeploy + manual approvals + Lambda steps.

## What it is

A directed-acyclic pipeline runner. Each **stage** has **actions** (parallel or sequential). Actions can be from CodeBuild, CodeDeploy, Lambda, manual approval, CloudFormation, third parties (GitHub, Jenkins, etc.).

## Key concepts

- **Pipeline** — top-level workflow.
- **Stage** — Source / Build / Test / Deploy / Approve.
- **Action** — operation within a stage.
- **Artifact** — produced by one action, consumed by next; stored in S3.
- **Pipeline types:**
  - **V1** — original.
  - **V2** — newer, triggers + variables + better fan-in.

## Real-world example

> ShareDeal release pipeline:
> 1. **Source** — GitHub push to `main`.
> 2. **Build** — CodeBuild runs tests, builds Docker image, pushes to ECR.
> 3. **Deploy to Staging** — CodeDeploy → ECS staging service.
> 4. **Manual Approval** — Slack notif; PM clicks Approve.
> 5. **Deploy to Prod** — CodeDeploy blue/green to ECS prod.

## Usage

CodePipeline templates are large; CDK or CFN strongly recommended. Skeleton:

```yaml
Resources:
  Pipeline:
    Type: AWS::CodePipeline::Pipeline
    Properties:
      RoleArn: !GetAtt PipelineRole.Arn
      ArtifactStore: { Type: S3, Location: !Ref ArtifactBucket }
      Stages:
        - Name: Source
          Actions:
            - Name: GitHub
              ActionTypeId: { Category: Source, Owner: AWS, Provider: CodeStarSourceConnection, Version: 1 }
              Configuration:
                ConnectionArn: !Ref GitHubConnection
                FullRepositoryId: arafatomer66/sd-app
                BranchName: main
              OutputArtifacts: [{ Name: SourceArt }]
        - Name: Build
          Actions:
            - Name: BuildAndPush
              ActionTypeId: { Category: Build, Owner: AWS, Provider: CodeBuild, Version: 1 }
              InputArtifacts: [{ Name: SourceArt }]
              OutputArtifacts: [{ Name: BuildArt }]
              Configuration: { ProjectName: !Ref BuildProject }
        - Name: Deploy
          Actions:
            - Name: DeployECS
              ActionTypeId: { Category: Deploy, Owner: AWS, Provider: ECS, Version: 1 }
              InputArtifacts: [{ Name: BuildArt }]
              Configuration:
                ClusterName: prod
                ServiceName: api
                FileName: imagedefinitions.json
```

## CDK Pipelines (much nicer)

```ts
const pipeline = new pipelines.CodePipeline(this, "Pipeline", {
  synth: new pipelines.ShellStep("Synth", {
    input: pipelines.CodePipelineSource.connection("arafatomer66/sd-infra", "main", { connectionArn: githubArn }),
    commands: ["npm ci", "npx cdk synth"],
  }),
});
pipeline.addStage(new MyStage(this, "Staging"));
pipeline.addStage(new MyStage(this, "Prod"), { pre: [new pipelines.ManualApprovalStep("Approve")] });
```

## Pricing

- **$1.00 per active pipeline per month** (V1).
- V2: tiered, slightly different.
- Free first pipeline.
- Plus actions' costs (CodeBuild minutes, etc.).

## CodePipeline vs GitHub Actions

- **CodePipeline** — multi-account/region orchestration via CDK Pipelines is a sweet spot. Lots of AWS-native actions.
- **GitHub Actions** — simpler for code-level CI. Use it + OIDC to AWS for deploys.

Many teams: **Actions for build/test, CodePipeline for cross-account deploys**.

## Gotchas

- **Artifact passing via S3 bucket** — bucket needs lifecycle to expire old artifacts.
- **Manual approval timeouts** default to 7 days.
- **Cross-account deploys** require IAM trust setup.
- **V1 pipeline updates** sometimes don't pick up new GitHub branches; recreate or re-link.

## Related

- [CodeBuild](./codebuild.md)
- [CodeDeploy](./codedeploy.md)
- [CDK Pipelines](./cdk.md)
