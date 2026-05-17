# Route 53

**TL;DR** — AWS's DNS service + domain registrar + health-check + routing-policy engine. 100% SLA. Global service.

## What it is

DNS at scale, with intelligent routing policies (latency-based, geolocation, weighted, failover). Can register domains too. Powers many of the world's largest sites.

## Key concepts

- **Hosted zone** — DNS records for one domain (e.g. `myapp.com`).
- **Record** — A, AAAA, CNAME, MX, TXT, SRV, NS, alias, etc.
- **Alias record** — Route 53-specific. Acts like CNAME but works at apex (`myapp.com`, not just `www.myapp.com`), and is free for AWS resources (ALB, CloudFront, S3 website, API Gateway).
- **Routing policies:**
  - **Simple** — single value.
  - **Weighted** — split traffic by % (canary).
  - **Latency-based** — route to lowest-latency region.
  - **Geolocation** — route by user's country.
  - **Geoproximity** — route by user's distance.
  - **Failover** — primary/secondary with health checks.
  - **Multivalue** — like simple but returns multiple healthy answers.
- **Health check** — HTTP/HTTPS/TCP probe to mark records up/down.
- **Private hosted zone** — DNS only resolvable inside a VPC.
- **Route 53 Resolver** — hybrid DNS in/out of VPC.

## Real-world example

> ShareDeal serves users from two regions:
> - `selefe.com` → Route 53 latency-based policy:
>   - `ap-south-1` ALB for South Asia users.
>   - `ap-southeast-1` ALB for SEA users.
> - Failover: if Mumbai health check fails, traffic auto-routes to Singapore.
> - `www.selefe.com` and `api.selefe.com` are aliases to CloudFront / ALB.

## Usage

### Register a domain

Console → Route 53 → Domains → Register. Or transfer in.

### Create hosted zone + records

```bash
aws route53 create-hosted-zone --name myapp.com --caller-reference $(date +%s)

# Update name servers at your registrar to those returned

aws route53 change-resource-record-sets --hosted-zone-id Z123ABC \
  --change-batch '{
    "Changes":[{
      "Action":"UPSERT",
      "ResourceRecordSet":{
        "Name":"api.myapp.com",
        "Type":"A",
        "AliasTarget":{
          "DNSName":"my-alb-xxxx.ap-south-1.elb.amazonaws.com",
          "HostedZoneId":"ZP97RAFLXTNZK",
          "EvaluateTargetHealth":true
        }
      }
    }]
  }'
```

### Health check

```bash
aws route53 create-health-check --caller-reference $(date +%s) \
  --health-check-config '{
    "Type":"HTTPS","FullyQualifiedDomainName":"api.myapp.com",
    "ResourcePath":"/health","Port":443,"RequestInterval":30,"FailureThreshold":3
  }'
```

### Latency-based + failover (record set example)

```json
[
  { "SetIdentifier":"mumbai",    "Region":"ap-south-1",     "Type":"A", "AliasTarget": { ... mumbai ALB ... }, "HealthCheckId": "hc-mumbai" },
  { "SetIdentifier":"singapore", "Region":"ap-southeast-1", "Type":"A", "AliasTarget": { ... singapore ALB ... }, "HealthCheckId": "hc-sg" }
]
```

Route 53 picks the lowest-latency healthy region.

## Pricing

- **Hosted zone:** $0.50/mo per zone.
- **Queries:** $0.40 per million standard, $0.60 latency-based.
- **Health checks:** $0.50/mo per check.
- **Domain registration:** ~$12/yr `.com`, varies by TLD.

Total for a small app: a few dollars/month.

## Gotchas

- **TTL matters.** Set TTL low (60s) for records that may failover; long (3600s) for stable records to reduce query cost.
- **DNS caching outside Route 53.** Browsers, resolvers, /etc/hosts all cache — failover isn't instant for end users.
- **Alias records don't show in `dig CNAME`.** They resolve directly to A/AAAA.
- **Geolocation records can miss countries** — set a default.
- **Private hosted zones** require `enableDnsHostnames` + `enableDnsSupport` on the VPC.
- **DNSSEC** supported (since 2020) but requires extra setup.

## Related

- [CloudFront](./cloudfront.md) — common alias target
- [ALB](./elb.md)
- [ACM](../05-security-iam/acm.md) — TLS certs, often validated via Route 53 DNS records
