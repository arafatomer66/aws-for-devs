# Regions & Service Availability

**TL;DR** — 30+ regions globally. Pick by latency, compliance, cost, and service availability. New services usually launch in `us-east-1` first.

## Major regions

| Code | Name | Where | Why you'd pick it |
|---|---|---|---|
| `us-east-1` | N. Virginia | USA East | Cheapest, most services, all global services live here |
| `us-east-2` | Ohio | USA East | Backup for us-east-1 |
| `us-west-2` | Oregon | USA West | Cheap, sustainable, good for ML |
| `eu-west-1` | Ireland | EU | First EU region, mature |
| `eu-west-2` | London | EU | UK data residency |
| `eu-central-1` | Frankfurt | EU | DACH region |
| `ap-south-1` | Mumbai | India | South Asia, **closest to Bangladesh** |
| `ap-south-2` | Hyderabad | India | Newer Indian region |
| `ap-southeast-1` | Singapore | SE Asia | SEA hub |
| `ap-southeast-2` | Sydney | Australia | ANZ |
| `ap-northeast-1` | Tokyo | Japan | Japan |
| `ap-east-1` | Hong Kong | China-adjacent | HK + South China |
| `me-central-1` | UAE | Middle East | GCC |
| `sa-east-1` | São Paulo | Brazil | LATAM |
| `af-south-1` | Cape Town | Africa | Africa |
| `cn-north-1` / `cn-northwest-1` | China | Operated by partners (BJS/NWCD) — **separate accounts** |
| GovCloud (US-East, US-West) | USA | Government workloads only |

## Service availability

**Not every service is in every Region.** Examples:
- **Bedrock** — limited to ~10 regions (us-east-1, us-west-2, ap-south-1, etc.).
- **CodeCommit** — being deprecated in some regions (use GitHub).
- **SageMaker** features lag behind in newer regions.
- **Local Zones / Wavelength** — extension zones inside specific metros.

Check current availability: **`aws.amazon.com/about-aws/global-infrastructure/regional-product-services/`**

CLI check:
```bash
# Is Bedrock in ap-south-1?
aws bedrock list-foundation-models --region ap-south-1
```

## Global vs Regional services

**Global services** (no region required):
- IAM
- Route 53
- CloudFront
- WAF (for CloudFront distributions)
- AWS Organizations
- Trusted Advisor
- Health Dashboard

**Regional services** — most others. Resources in `us-east-1` are invisible to `ap-south-1` unless you cross-region replicate.

## Picking a region for your project

Decision flow:

1. **Compliance / data residency?** Must be in a specific country → forced choice.
2. **Where are your users?** Pick closest region for p99 latency.
3. **Service available?** Confirm.
4. **Cost?** `us-east-1` < most others.
5. **Disaster recovery?** Plan a secondary region (usually same continent, different region).

## us-east-1 is special

- All **global services** are physically rooted here (IAM, Route 53, etc.).
- New services launch here first.
- It's the **largest, oldest, and cheapest** region.
- But it's also had **the most outages**. Don't make your "primary" region your only region.

## Real-world example

> For a Bangladesh-targeted app like ShareDeal:
>
> - **Primary:** `ap-south-1` (Mumbai). ~30–60 ms RTT from Dhaka, all major services available.
> - **DR (disaster recovery):** `ap-southeast-1` (Singapore). Slightly farther, but if Mumbai is down, you have a hot standby.
> - **Edge:** CloudFront automatically uses Dhaka/Singapore PoPs.

## Useful CLI tricks

```bash
# List all regions
aws ec2 describe-regions --output table

# Where do my EC2 instances live?
for r in $(aws ec2 describe-regions --query 'Regions[].RegionName' --output text); do
  echo "=== $r ==="
  aws ec2 describe-instances --region $r --query 'Reservations[].Instances[].InstanceId'
done

# Current region for current session
aws configure get region
```

## Gotchas

- **GovCloud and China regions need separate AWS accounts.** They're not in your normal account.
- **AZ names aren't stable across accounts.** `us-east-1a` in your account may be `us-east-1b` in mine. Use AZ IDs for consistency.
- **Some pricing is region-specific.** Same EC2 instance can be 20% more in `sa-east-1` than `us-east-1`.
- **Cross-region API calls add latency** — keep app + DB in same region.

## Related

- [Global Infrastructure](./01-global-infrastructure.md)
- [Accounts & Billing](./04-accounts-billing.md)
