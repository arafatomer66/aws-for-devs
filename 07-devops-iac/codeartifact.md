# CodeArtifact

**TL;DR** — Managed artifact repository. Hosts npm / PyPI / Maven / NuGet / generic packages. Acts as a proxy to public registries with caching + auth.

## What it is

A managed package repo. Two purposes:
1. **Internal package hosting** — your private @company/* npm or `com.company.*` Maven packages.
2. **Proxy to public registries** — pull from npmjs / PyPI / Maven Central, cache them, control versions.

## Key concepts

- **Domain** — top-level resource (one per org usually).
- **Repository** — within a domain.
- **Upstream** — pull from public registry through CodeArtifact.
- **Permissions:** IAM + repository policy.

## Real-world example

> A company:
> - Domain: `acme`.
> - Repos: `npm-store`, `pypi-store`, `internal-npm`, `internal-maven`.
> - `npm-store` proxies npmjs; cached versions appear locally.
> - `internal-npm` hosts `@acme/ui`, `@acme/utils`.
> - CI uses `aws codeartifact get-authorization-token` to fetch creds, then `npm install`.

## Usage

### Create domain + repo

```bash
aws codeartifact create-domain --domain acme
aws codeartifact create-repository --domain acme --repository npm-store \
  --upstreams repositoryName=npm-public
aws codeartifact associate-external-connection --domain acme --repository npm-store \
  --external-connection public:npmjs
```

### Configure npm to use it

```bash
aws codeartifact login --tool npm --domain acme --repository npm-store --region ap-south-1
# Sets registry + auth token in .npmrc for 12 hours
```

### Publish

```bash
npm publish      # to the registry configured above
```

### Pricing

- **$0.05/GB-mo storage.**
- **$0.05 per 10,000 requests.**
- **Data transfer** standard rates.

Typical small org: a few dollars/month.

## CodeArtifact vs alternatives

- **GitHub Packages** — if your code lives in GitHub, often simpler.
- **Verdaccio / Nexus** — self-hosted.
- **JFrog Artifactory** — enterprise, multi-format.

## Gotchas

- **Auth token expires every 12 hours** by default — re-run `login` in CI.
- **Doesn't host Docker images** — use ECR.
- **Doesn't replace SCM** — still need GitHub etc. for source.

## Related

- [CodeBuild](./codebuild.md)
- [ECR](#) — for container images
