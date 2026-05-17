# Shield — DDoS Protection

**TL;DR** — DDoS mitigation. Standard is free and on by default. Advanced is $3,000/mo and adds attack visibility, response team, and cost protection.

## Shield Standard

- **Always on, free, automatic.**
- Protects against most common Layer 3/4 attacks (SYN/UDP floods, reflection).
- Built into CloudFront, Route 53, Global Accelerator, ELB.
- You don't configure it; it just works.

## Shield Advanced

- **$3,000/mo per organization** + data fee.
- **Detailed attack diagnostics** in real time.
- **24/7 access to the Shield Response Team (SRT).**
- **Cost protection** — if a DDoS causes auto-scaling cost spikes, AWS may credit you.
- **WAF + Firewall Manager included.**
- **Layer 7 protection** with WAF auto-mitigation.
- **Protects:** CloudFront, ELB, Global Accelerator, Route 53, EIPs.

## When to consider Shield Advanced

- High-stakes consumer-facing site (banks, e-commerce, gaming).
- Past DDoS incidents.
- Compliance or insurance requires it.
- You'd rather pay $36k/year than risk a 3-hour outage.

For most startups: **Standard is fine**. WAF + CloudFront handle most volumetric stuff.

## Real-world example

> A betting platform during the World Cup gets repeated 500 Gbps attacks. They:
> - Subscribe to Shield Advanced.
> - SRT pre-configures mitigations during peak hours.
> - WAF rules auto-tighten under attack.
> - Costs from auto-scaling absorbed by cost protection.

## Usage

```bash
# Subscribe (one-time, account-wide)
aws shield create-subscription

# Add a resource for protection
aws shield create-protection --name api-elb \
  --resource-arn arn:aws:elasticloadbalancing:ap-south-1:..:loadbalancer/app/api/...
```

## Gotchas

- **Subscription is per-org, billed monthly, year-minimum.** Not a "turn on for a week" thing.
- **WAF still needs configuration** — Shield Advanced doesn't auto-block L7 unless rules tell it to.
- **EIPs need explicit protection** — they're not protected by default.

## Related

- [WAF](./waf.md)
- [CloudFront](../04-networking/cloudfront.md)
- [Global Accelerator](../04-networking/global-accelerator.md)
