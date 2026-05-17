# AWS Global Infrastructure

**TL;DR** — AWS physically lives in **Regions** (geographic areas) that contain **Availability Zones** (clusters of data centers). Pick the right Region close to your users and spread your workload across multiple AZs for resilience.

## The hierarchy

```
AWS Cloud
├── Region (e.g. us-east-1, ap-south-1, eu-west-2)
│   ├── Availability Zone (e.g. us-east-1a, us-east-1b, us-east-1c)
│   │   └── Data Center (one or more buildings, fiber, power)
├── Edge Location (300+ globally — CloudFront, Route 53)
├── Local Zone (extension of a Region for ultra-low latency in metro areas)
├── Wavelength Zone (inside telco 5G networks)
└── Outpost (AWS hardware in your own data center)
```

## Regions

A Region is a **physical geographic area** with multiple isolated data centers. Examples: `us-east-1` (N. Virginia), `eu-west-1` (Ireland), `ap-southeast-1` (Singapore), `ap-south-1` (Mumbai — closest to Bangladesh).

- **Region codes** are like `us-east-1`, `eu-west-2`, `ap-south-1`. You'll type these a LOT.
- Most services are **regional** — resources in `us-east-1` can't directly see resources in `eu-west-1`.
- A few services are **global**: IAM, Route 53, CloudFront, WAF (for CloudFront), Organizations.
- Not every service is in every Region. Newer services often launch in `us-east-1` first.
- Pricing varies by Region. `us-east-1` is usually cheapest.

**How to choose a region:**
1. **Latency** — closest to your users.
2. **Compliance** — data residency laws (GDPR for EU, etc).
3. **Service availability** — some services aren't everywhere.
4. **Cost** — `us-east-1` < `ap-south-1` < `sa-east-1` for many services.

## Availability Zones (AZs)

An AZ is a **cluster of one or more data centers** inside a Region, with redundant power/cooling/networking.

- AZs in the same Region are tens of kilometers apart — close enough for low-latency replication (~1ms), far enough that a flood or fire doesn't take them all out together.
- AZ names like `us-east-1a` are **mapped per-account** — your `us-east-1a` may not be my `us-east-1a` physically. Use AZ IDs (`use1-az1`) when you need consistency across accounts.
- **Rule of thumb: always deploy to ≥ 2 AZs.** A single AZ outage shouldn't kill your app.

## Edge Locations / Points of Presence (PoPs)

Hundreds of small sites globally that **cache content and terminate connections close to users**. Used by:
- **CloudFront** (CDN)
- **Route 53** (DNS)
- **Global Accelerator** (anycast IPs)
- **AWS WAF** (when paired with CloudFront)
- **API Gateway** (edge-optimized endpoints)

You don't pick these directly — AWS routes the user to the nearest edge.

## Local Zones

Mini-regions in big metros (LA, Boston, Dhaka, etc.) for **single-digit-ms latency** workloads like gaming, live media, and real-time finance. Compute + storage only; not every service is there.

## Wavelength Zones

AWS infrastructure **inside a telco's 5G network**. For ultra-low-latency mobile apps (AR/VR, robotics).

## Outposts

A rack of AWS hardware **shipped to your own data center**, managed by AWS, looking like a regional extension. For latency- or compliance-bound workloads that must stay on-prem.

## Real-world example

> You're building **ShareDeal** (group-buying app, Bangladesh users).
>
> - Pick **`ap-south-1` (Mumbai)** — closest to BD users, ~30 ms RTT.
> - Run your RDS Aurora cluster across **`ap-south-1a` + `ap-south-1b` + `ap-south-1c`** (Multi-AZ).
> - Serve product images via **CloudFront** — edge locations in Dhaka cache them locally.
> - Use **Route 53** for DNS — global service, automatically resolves to nearest CloudFront edge.

## Gotchas

- **Don't forget the region in CLI calls.** `aws s3 ls` defaults to your configured region; cross-region commands need `--region`.
- **AZ IDs vs. AZ names** — `us-east-1a` is not stable across accounts. Use `aws ec2 describe-availability-zones` to see both.
- **us-east-1 is special** — IAM and many global services are physically run from there. When `us-east-1` has a major outage, the AWS Console itself can break.
- **Data transfer between Regions costs money** (≈$0.02/GB). Within a Region, between AZs costs ~$0.01/GB. Within an AZ is free.

## Related

- [Regions & Service Availability](./06-regions.md)
- [VPC](../04-networking/vpc.md) — how networks span AZs
- [Route 53](../04-networking/route53.md)
- [CloudFront](../04-networking/cloudfront.md)
