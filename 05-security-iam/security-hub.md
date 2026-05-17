# Security Hub

**TL;DR** — Single pane of glass for security findings + compliance posture across accounts. Aggregates GuardDuty, Inspector, Config, Macie, IAM Access Analyzer, third parties.

## What it is

A central aggregator + scorer for security signals. Runs continuous compliance checks against standards (CIS, PCI DSS, NIST, AWS FSBP) and ingests findings from other security services into a unified format (ASFF — AWS Security Finding Format).

## Key concepts

- **Standard** — control framework: AWS Foundational Best Practices, CIS, PCI DSS, NIST 800-53.
- **Control** — individual rule (e.g. "Root access keys disabled").
- **Finding** — instance of a control failure or signal.
- **Insight** — saved filter/group of findings (e.g. all crit findings unresolved).
- **Automation rule** — auto-triage findings (assign, suppress, change severity).
- **Delegated admin** — central account that sees findings from all member accounts.

## Real-world example

> Multi-account org enables Security Hub via Organizations:
> - GuardDuty, Inspector, Config, IAM Access Analyzer all turned on in each account.
> - Findings flow to the central security account.
> - SOC dashboard: open critical findings count, MTTR, compliance score per standard.

## Usage

```bash
# Enable Security Hub
aws securityhub enable-security-hub --enable-default-standards

# List standards
aws securityhub describe-standards

# Get findings
aws securityhub get-findings --max-results 10
```

EventBridge integration: trigger on `Security Hub Findings - Imported` for alerting.

## Pricing

- ~$0.0010 per finding ingested + $0.0030 per compliance check (per check, per resource per month).
- Typical: $50-200/mo for a serious organization.

## Gotchas

- Multi-region: enable everywhere or be blind in unenabled regions.
- Costs scale with resource count and finding volume.
- Findings aren't fixes — pair with automation rules + remediation Lambdas.

## Related

- [GuardDuty](./guardduty.md)
- [Inspector](#)
- [Config](../08-monitoring-observability/config.md)
