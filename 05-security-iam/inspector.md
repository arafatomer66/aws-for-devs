# Amazon Inspector

**TL;DR** — Vulnerability scanner for EC2, ECR container images, and Lambda. Continuous, CVE-based, integrates with Security Hub.

## What it is

A managed vulnerability assessment service. Continuously scans:
- **EC2 instances** — OS + installed packages.
- **ECR images** — base layers + application dependencies (Inspector v2 / "Enhanced scanning").
- **Lambda functions** — code + dependencies.

Findings are CVE-scored with severity, exploitability, network reachability context.

## Key concepts

- **Inspector v2** is the current product (don't confuse with the old "Inspector Classic" which is being retired).
- **Findings** — per-resource CVE list with CVSS scores.
- **Network reachability** — EC2 findings include "is this exposed to the internet?" context.
- **Suppression rules** — silence known-acceptable findings.
- **Integration** — Security Hub, EventBridge, ECR (block pulls of bad images).

## Real-world example

> CI/CD gate on container vulnerabilities:
> - `docker push` to ECR triggers Inspector scan.
> - Finding: Critical CVE in `openssl` base layer.
> - EventBridge rule → Lambda → fails the CodePipeline stage.
> - Team rebuilds with patched base; redeploys.

## Usage

### Enable

```bash
aws inspector2 enable --resource-types EC2 ECR LAMBDA
```

### List findings

```bash
aws inspector2 list-findings \
  --filter-criteria '{"severity":[{"comparison":"EQUALS","value":"CRITICAL"}]}'
```

### EventBridge integration

```json
{
  "source": ["aws.inspector2"],
  "detail-type": ["Inspector2 Finding"],
  "detail": { "severity": ["CRITICAL", "HIGH"] }
}
```

Pipe to Slack / PagerDuty / a Lambda that auto-creates Jira tickets.

## Pricing

Per-resource per-month:
- EC2: $1.258 per instance / month.
- ECR initial scan: $0.09 per image scanned.
- ECR continual rescan: $0.01 per image per month.
- Lambda: $0.30 per function / month.

For a 10-EC2 + 50-image + 30-Lambda account: ~$30/mo.

## Inspector vs GuardDuty vs Macie

- **Inspector** — *finds vulnerabilities* (CVEs) in your resources.
- **GuardDuty** — *finds threats* (suspicious behavior in logs).
- **Macie** — *finds sensitive data* (PII in S3).

All three are complementary; turn them on together for serious workloads.

## Gotchas

- **EC2 scanning needs SSM Agent + instance role.** Without it, no findings.
- **Initial scan on enable is slow** (hours for big accounts).
- **Suppression rules can hide useful findings** — review them periodically.
- **Lambda scanning** scans the deployed package; if you store secrets in env vars, encrypt them.

## Related

- [Security Hub](./security-hub.md)
- [GuardDuty](./guardduty.md)
- [Macie](./macie.md)
- [ECR](../01-compute/ecr.md)
