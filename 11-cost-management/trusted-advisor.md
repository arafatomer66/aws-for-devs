# Trusted Advisor

**TL;DR** — Automated best-practice checks across cost, security, fault tolerance, performance, service limits. Some free, more with paid support plans.

## What it is

A health-check service that scans your account and produces actionable recommendations. Checks group into 5 pillars:
- Cost optimization (idle resources, underutilized).
- Performance.
- Security (open SGs, MFA on root, public buckets).
- Fault tolerance (multi-AZ, snapshot age).
- Service limits.

## Tiers

- **Basic / Developer Support:** ~6 core checks free (mostly security).
- **Business / Enterprise Support:** full suite (100+ checks).

If you have Business support, **run Trusted Advisor monthly.** It pays for itself.

## Real-world example

> Trusted Advisor flags:
> - 12 unattached EBS volumes ≈ $40/mo wasted.
> - 3 idle RDS instances.
> - 2 security groups with `0.0.0.0/0` to port 22.
> - Multi-AZ off on a critical RDS.
>
> Fix list for next on-call rotation.

## Usage

Console: AWS Support → Trusted Advisor.

API:
```bash
aws support describe-trusted-advisor-checks --language en --region us-east-1
aws support describe-trusted-advisor-check-result --check-id <id>
```

(Support API is `us-east-1` only.)

## Gotchas

- **Some checks need Business/Enterprise support** — verify your plan.
- **Service limit warnings** are very useful — you'll hit limits unexpectedly otherwise.
- **Not real-time** — refreshes periodically.

## Related

- [Cost Explorer](./cost-explorer.md)
- [Compute Optimizer](./compute-optimizer.md)
- [Config](../08-monitoring-observability/config.md)
- [Security Hub](../05-security-iam/security-hub.md)
