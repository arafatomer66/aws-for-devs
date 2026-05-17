# GuardDuty — Threat Detection

**TL;DR** — ML-driven threat detection on CloudTrail logs, VPC Flow Logs, DNS, EKS audit, S3 data events, RDS login events. Turn it on; get findings.

## What it is

A managed threat-detection service. Continuously analyzes log streams for known + ML-detected attacker patterns (crypto-mining, unusual API calls, port scans, credential theft, exfiltration).

## Key concepts

- **Detector** — the GuardDuty service instance per account+region.
- **Finding** — a specific suspicious event. Severity Low/Medium/High.
- **Data sources** — CloudTrail mgmt + S3 events, VPC Flow Logs, DNS logs, EKS audit, RDS login (Aurora), Lambda network activity, EBS malware scan.
- **Threat intel** — AWS + partners (Proofpoint, CrowdStrike).
- **Auto-archive rules** — suppress known-noise findings.

## Real-world example

> A leaked IAM access key. GuardDuty notices:
> - `Recon:IAMUser/UserPermissions` — caller listing IAM resources from a new IP.
> - `UnauthorizedAccess:IAMUser/MaliciousIPCaller` — calls from a known TOR exit.
> - SOC team gets a Slack alert via EventBridge → Lambda → Slack webhook.
> - Rotate keys, set conditions on the IAM user, investigate damage.

## Usage

### Enable

```bash
aws guardduty create-detector --enable
```

### List findings

```bash
aws guardduty list-detectors
aws guardduty list-findings --detector-id <id>
aws guardduty get-findings --detector-id <id> --finding-ids <id-1> <id-2>
```

### Wire to EventBridge for alerts

EventBridge rule pattern:
```json
{
  "source": ["aws.guardduty"],
  "detail-type": ["GuardDuty Finding"],
  "detail": { "severity": [{ "numeric": [">=", 4.0] }] }
}
```
Target: SNS topic / Slack Lambda / PagerDuty.

## Pricing

Pricing varies per data source. Typical small account: $20-50/mo. Big accounts (lots of CloudTrail events) can be $100s. Worth it.

## Gotchas

- **Per-region.** Enable in every region you use (or at least every region where you have resources).
- **Initial week** — baselining; findings may be noisy.
- **Doesn't fix anything**, just detects. Pair with runbooks or auto-remediation Lambdas.
- **Multi-account** — use Organizations to enable cluster-wide and centralize findings.

## Related

- [Security Hub](./security-hub.md) — aggregates GuardDuty + Inspector + Config findings
- [Inspector](#) — vulnerability scanner for EC2/ECR/Lambda
- [CloudTrail](../08-monitoring-observability/cloudtrail.md)
