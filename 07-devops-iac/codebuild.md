# CodeBuild

**TL;DR** — Managed build/CI runner. Pay-per-minute. Runs your `buildspec.yml` in a container. Good for AWS-native CI; many teams now use GitHub Actions → AWS via OIDC instead.

## What it is

A managed container that runs commands on every trigger (git push, manual, schedule). Pulls source, runs `buildspec.yml`, optionally pushes artifacts (to S3, ECR, etc.).

## Key concepts

- **Project** — your CI config.
- **Source** — GitHub / GitLab / Bitbucket / S3 / CodeCommit.
- **Environment** — Docker image to build inside (AWS-curated `aws/codebuild/standard:7.0` or custom).
- **Compute type** — `BUILD_GENERAL1_SMALL/MEDIUM/LARGE/2XLARGE` + GPU + ARM.
- **buildspec.yml** — what to run, in phases (install / pre_build / build / post_build).
- **Artifacts** — files to ship out (zip to S3, push to ECR).
- **Cache** — local cache or S3 cache.
- **Webhook** — trigger on git push.

## buildspec.yml example

```yaml
version: 0.2
env:
  variables:
    NODE_OPTIONS: "--max_old_space_size=4096"
phases:
  install:
    runtime-versions: { nodejs: 20 }
    commands:
      - npm ci
  build:
    commands:
      - npm run lint
      - npm test
      - npm run build
      - docker build -t my-app .
  post_build:
    commands:
      - aws ecr get-login-password | docker login --username AWS --password-stdin $ECR_REGISTRY
      - docker tag my-app $ECR_REGISTRY/my-app:$CODEBUILD_RESOLVED_SOURCE_VERSION
      - docker push $ECR_REGISTRY/my-app:$CODEBUILD_RESOLVED_SOURCE_VERSION
artifacts:
  files:
    - "**/*"
  base-directory: dist
cache:
  paths:
    - node_modules/**/*
```

## Usage

```bash
aws codebuild create-project \
  --name sd-build \
  --source 'type=GITHUB,location=https://github.com/arafatomer66/sd-app.git,reportBuildStatus=true' \
  --artifacts 'type=NO_ARTIFACTS' \
  --environment 'type=LINUX_CONTAINER,image=aws/codebuild/standard:7.0,computeType=BUILD_GENERAL1_MEDIUM,privilegedMode=true' \
  --service-role arn:aws:iam::..:role/codebuild-role

aws codebuild start-build --project-name sd-build
```

Or wire to a webhook so every PR/push runs builds.

## CodeBuild vs GitHub Actions

| | CodeBuild | GitHub Actions |
|---|---|---|
| Cost | $0.005/min `general1.small` ≈ $0.30/hr | Free for public, generous tiers for private |
| AWS integration | Native (roles, ECR, S3) | Via OIDC (no static keys, recommended) |
| Speed | Decent | Decent |
| Ecosystem | Smaller | Huge marketplace |
| Best for | Pure AWS shops | Most modern teams |

**Trend:** GitHub Actions → OIDC → AssumeRole into AWS is more popular for new projects.

## Pricing

- $0.005/min for `general1.small`, $0.01 for medium, $0.02 for large.
- $0.05/min for `2xlarge`.
- ARM Graviton2 instances ~20% cheaper.

A 5-min build × 30 builds/day × 30 days = 75 min/day = ~$11/mo.

## Gotchas

- **Docker in Docker** — set `privilegedMode: true` to build images.
- **Cache to S3** for big `node_modules` — saves minutes per build.
- **Default timeout 60 min**, max 8 hours.
- **Environment variables for secrets** — use Parameter Store / Secrets Manager references, not plain text.
- **Cold start** — first build of the day takes longer; warm container pool is internal.

## Related

- [CodePipeline](./codepipeline.md)
- [CodeDeploy](./codedeploy.md)
- [CodeArtifact](./codeartifact.md)
- [ECR](#) — push image targets
