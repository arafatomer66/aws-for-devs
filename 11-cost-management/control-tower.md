# Control Tower

**TL;DR** — AWS's opinionated "landing zone" for multi-account orgs. Sets up Organizations + logging + guardrails + account factory in one click.

## What it is

A managed multi-account framework. Control Tower provisions:
- **Log Archive account** + central S3 bucket for CloudTrail / Config logs.
- **Audit/Security account** for findings (GuardDuty, Security Hub).
- **IAM Identity Center** with default permission sets.
- **Account Factory** — provision new accounts with baseline guardrails.
- **Controls (guardrails)** — preventive (SCPs) + detective (Config rules) + proactive (CFN hooks).
- **Customizations for Control Tower (CfCT)** — extension framework.

## Why use it

If you're going to build a multi-account AWS org anyway, Control Tower saves weeks. Comes with reasonable defaults; you can extend or override.

## Real-world example

> Mid-size company spinning up AWS:
> - Launch Control Tower in the management account.
> - Get `Log Archive`, `Audit`, `Network`, and OU structure created.
> - Enable ~30 default guardrails.
> - Account Factory provisions new dev/prod accounts pre-baselined.

## Pricing

- **Control Tower itself is free** for the orchestration.
- You pay for resources it creates (CloudTrail, Config in each account, S3 storage for logs).

## Gotchas

- **Hard to undo** — leaving Control Tower partially is messy. Decide before adopting.
- **Customization is via CfCT / CFN/CDK** — feels heavier than terraform-native setups.
- **Some guardrails are mandatory** — can't disable them.
- **Re-baselining** — periodically Control Tower wants to re-apply baseline; can disrupt resources in OUs.

## Control Tower vs DIY

| | Control Tower | DIY (Org + CDK/Terraform) |
|---|---|---|
| Speed | Hours | Weeks |
| Flexibility | Lower | Total |
| Best for | Companies wanting AWS-blessed defaults | Teams with strong opinions |

## Related

- [Organizations](./organizations.md)
- [Identity Center](../05-security-iam/identity-center.md)
- [Config](../08-monitoring-observability/config.md)
- [Security Hub](../05-security-iam/security-hub.md)
