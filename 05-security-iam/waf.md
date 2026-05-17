# WAF — Web Application Firewall

**TL;DR** — Rule-based HTTP filtering attached to CloudFront, ALB, API Gateway, AppSync, App Runner. Blocks SQLi/XSS/bots/rate-spam/specific IPs/etc.

## What it is

A web application firewall as a service. You write rules (or pick managed rule groups) and attach them to a "Web ACL" tied to a CloudFront/ALB/API GW.

## Key concepts

- **Web ACL** — the firewall instance.
- **Rule** — match condition + action.
- **Match conditions:** IP set, geo, headers, body, URI, query string, regex, SQLi/XSS pattern, rate-based, JA3 fingerprint.
- **Actions:** Allow, Block, Count, CAPTCHA, Challenge.
- **Rule group** — bundle of rules (own or AWS Managed).
- **Managed Rules** — AWS-curated and Marketplace vendor rules (F5, Imperva, Fortinet).

## Common managed rule groups

- **`AWSManagedRulesCommonRuleSet`** — OWASP-ish, the baseline.
- **`AWSManagedRulesKnownBadInputs`** — log4shell, etc.
- **`AWSManagedRulesSQLiRuleSet`** — SQL injection patterns.
- **`AWSManagedRulesLinuxRuleSet`** / **WindowsRuleSet** / **POSIXRuleSet** — OS-specific exploits.
- **`AWSManagedRulesBotControlRuleSet`** — bot detection (more $$).
- **`AWSManagedRulesATPRuleSet`** — account takeover protection (login pages).
- **`AWSManagedRulesAmazonIpReputationList`** — known bad IPs.

## Real-world example

> ShareDeal API has been getting credential-stuffing attempts:
> - Attach `AWSManagedRulesATPRuleSet` to `/auth/login` path.
> - Add a rate-based rule: `> 100 reqs/5 min from one IP → Block`.
> - Set geo rule: only `BD, IN, AE` allowed for admin endpoints.

## Usage

### Create + attach (CLI, abridged)

```bash
aws wafv2 create-web-acl --scope REGIONAL --name api-waf \
  --default-action Allow={} \
  --rules file://rules.json \
  --visibility-config 'SampledRequestsEnabled=true,CloudWatchMetricsEnabled=true,MetricName=api-waf'

aws wafv2 associate-web-acl --web-acl-arn arn:... \
  --resource-arn arn:aws:elasticloadbalancing:ap-south-1:..:loadbalancer/app/api/...
```

### Rate-based rule (JSON snippet)

```json
{
  "Name": "rate-limit",
  "Priority": 10,
  "Action": { "Block": {} },
  "Statement": {
    "RateBasedStatement": {
      "Limit": 1000,
      "AggregateKeyType": "IP"
    }
  },
  "VisibilityConfig": { "SampledRequestsEnabled": true, "CloudWatchMetricsEnabled": true, "MetricName": "rate-limit" }
}
```

## Scopes

- **CLOUDFRONT** — global, region must be `us-east-1`.
- **REGIONAL** — for ALB / API Gateway / AppSync / App Runner / Cognito User Pool, region-specific.

## Pricing

- **Web ACL:** $5/mo.
- **Rule:** $1/mo per rule (managed groups count as 1 each).
- **Requests:** $0.60 per million.
- **Bot Control / ATP / Fraud Control:** extra per-request fees.

Typical baseline: $20-50/mo per Web ACL. Bot/ATP add-ons can run $100s.

## Gotchas

- **Rules evaluated in priority order, lowest first.** First Block or Allow wins.
- **Count action** is for testing — rule matches but doesn't block. Use before going live.
- **Body inspection size limit** — default 8 KB body. Increase if you need to inspect larger payloads.
- **Managed rules can false-positive** — start with Count, watch metrics, then switch to Block.
- **WAF doesn't inspect inside encrypted tunnels** other than the proxied protocols.
- **Geo block isn't a perfect VPN-stopper** — determined attackers route around it.

## Related

- [Shield](./shield.md) — DDoS, separate but complementary
- [CloudFront](../04-networking/cloudfront.md)
- [API Gateway](../04-networking/api-gateway.md)
- [ALB](../04-networking/elb.md)
