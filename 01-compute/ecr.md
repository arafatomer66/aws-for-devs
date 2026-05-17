# ECR — Elastic Container Registry

**TL;DR** — Private (and public) Docker registry. Tight IAM integration with ECS/EKS/Lambda. Image scanning + lifecycle policies. ~$0.10/GB-month.

## What it is

A managed container image registry. Two flavors:
- **ECR Private** — IAM-authenticated, regional.
- **ECR Public** — public registry at `public.ecr.aws/<alias>/<repo>`.

Compatible with the standard Docker CLI + OCI tooling. Used by every AWS container workflow.

## Why it exists

You need somewhere to push images. Docker Hub works but: rate limits, separate auth, sometimes flaky. ECR is in-region, IAM-auth'd, integrates seamlessly with ECS / EKS / Lambda / App Runner / CodeBuild.

## Key concepts

- **Repository** — one named image stream (`my-app`, `my-app/api`).
- **Image tag** — `latest`, `v1.2.3`, git SHA.
- **Image digest** — content-addressable SHA256 (immutable).
- **Lifecycle policy** — auto-expire old images (e.g., "keep last 30").
- **Image scanning** — basic (free, snapshot at push) or enhanced (Inspector, continuous).
- **Replication** — cross-region or cross-account auto-replication.
- **Pull-through cache** — proxy + cache for public registries (Docker Hub, GHCR, Quay, K8s).

## Real-world example

> ShareDeal pipeline:
> - CodeBuild builds image → tags `api:$GIT_SHA` + `api:latest`.
> - Pushes to ECR `123456789012.dkr.ecr.ap-south-1.amazonaws.com/api`.
> - ECS service task definition references the SHA tag for immutable deploys.
> - Lifecycle policy keeps last 30 images per repo; rest auto-expire.
> - Enhanced scanning (Inspector) finds CVEs; pipeline gates on Critical findings.

## Usage

### Create a repo

```bash
aws ecr create-repository \
  --repository-name api \
  --image-scanning-configuration scanOnPush=true \
  --encryption-configuration encryptionType=AES256
```

### Login + push (Docker CLI)

```bash
aws ecr get-login-password --region ap-south-1 \
  | docker login --username AWS --password-stdin 123456789012.dkr.ecr.ap-south-1.amazonaws.com

docker build -t api .
docker tag api:latest 123456789012.dkr.ecr.ap-south-1.amazonaws.com/api:v1.2.3
docker push           123456789012.dkr.ecr.ap-south-1.amazonaws.com/api:v1.2.3
```

### Pull (anywhere with IAM)

ECS / EKS / Lambda do this automatically given the right task/execution role. Locally:

```bash
docker pull 123456789012.dkr.ecr.ap-south-1.amazonaws.com/api:v1.2.3
```

### Lifecycle policy

```json
{
  "rules": [
    {
      "rulePriority": 1,
      "description": "Keep last 30 images",
      "selection": { "tagStatus": "any", "countType": "imageCountMoreThan", "countNumber": 30 },
      "action": { "type": "expire" }
    },
    {
      "rulePriority": 2,
      "description": "Expire untagged after 7 days",
      "selection": { "tagStatus": "untagged", "countType": "sinceImagePushed", "countUnit": "days", "countNumber": 7 },
      "action": { "type": "expire" }
    }
  ]
}
```

```bash
aws ecr put-lifecycle-policy --repository-name api --lifecycle-policy-text file://lifecycle.json
```

### Pull-through cache (for Docker Hub etc.)

```bash
aws ecr create-pull-through-cache-rule \
  --ecr-repository-prefix dockerhub --upstream-registry-url registry-1.docker.io
docker pull 123.dkr.ecr.ap-south-1.amazonaws.com/dockerhub/library/nginx:latest
```

ECR caches public images in your account; subsequent pulls skip the public internet.

## Pricing

- **Storage:** $0.10/GB-month.
- **Data transfer out to internet:** standard rates.
- **Within-region pulls:** free.
- **Cross-region replication:** standard cross-region transfer.
- **Public ECR:** free for OSS via the AWS OSS Sponsorship; otherwise small storage/transfer fees.

## Gotchas

- **Untagged images don't auto-delete** — set a lifecycle policy or accumulate orphans (and pay).
- **Use the digest (`@sha256:...`) for deploys**, not `latest`, for true immutable rollbacks.
- **Cross-account pulls** need both an IAM policy and a repository policy.
- **First push from Docker Hub upstream is slow** (cache miss).
- **ECR Public is in `us-east-1` only.**
- **Multi-arch images** — push manifest list with arm64 + amd64 for Graviton support.

## Related

- [ECS](./ecs.md)
- [EKS](./eks.md)
- [Lambda](./lambda.md) — supports container images
- [CodeBuild](../07-devops-iac/codebuild.md)
- [Inspector](../05-security-iam/inspector.md) — enhanced scanning
