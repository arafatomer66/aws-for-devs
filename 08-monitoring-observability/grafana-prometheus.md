# Amazon Managed Grafana & Managed Prometheus

**TL;DR** — Managed versions of Grafana and Prometheus. For teams that already love these tools and don't want to run them.

## Amazon Managed Service for Prometheus (AMP)

A managed Prometheus-compatible time-series store. Push metrics via remote_write; query with PromQL.

- No node sizing — auto-scales.
- Long-term storage (~150 days).
- Integrates with EKS via the ADOT collector or the Prometheus Operator.

### Pricing
- **$0.90 per 10M metric samples ingested.**
- **$0.03 per GB-mo storage.**
- **Query: $0.10 per billion samples processed.**

### Real-world example

> EKS cluster running prom-style exporters:
> - ADOT collector scrapes them, remote_writes to AMP.
> - Managed Grafana visualizes.
> - Alerts pipe to SNS → PagerDuty.

## Amazon Managed Grafana (AMG)

A managed Grafana workspace. Auth via IAM Identity Center, SAML, or AWS-native. Built-in plugins for CloudWatch, AMP, X-Ray, AWS IoT SiteWise, Athena, Timestream, OpenSearch, Redshift, Aurora — plus standard Grafana sources (Prometheus, Loki, etc.).

### Pricing
- **Editor:** $9/user/mo.
- **Viewer:** $5/user/mo.

### Real-world example

> SRE team uses Managed Grafana to combine CloudWatch metrics, AMP metrics, and X-Ray traces on the same dashboard. Identity via SSO; per-user billing.

## When AMP/AMG vs alternatives

- **AMP + AMG** — if you want Prometheus/Grafana operationally simpler than self-hosting.
- **CloudWatch** — if you don't have a Grafana habit.
- **Datadog / NewRelic / Honeycomb** — if you want a polished commercial product.

## Gotchas

- AMP doesn't run a Prometheus you can `kubectl exec` into — it's a remote_write target only.
- Per-user Grafana billing surprises small teams.
- Some Grafana plugins aren't supported in AMG.

## Related

- [CloudWatch](./cloudwatch.md)
- [X-Ray](./x-ray.md)
- [EKS](../01-compute/eks.md)
