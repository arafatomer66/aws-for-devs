# Global Accelerator

**TL;DR** — Two static anycast IPs that route users via AWS's backbone network to your endpoints. For non-HTTP traffic or HTTP that needs faster, more reliable routing than CloudFront.

## What it is

You get two **static anycast IPs** announced from edge PoPs worldwide. Users hit the nearest PoP; traffic then rides AWS's private backbone to your endpoints (ALB/NLB/EC2/EIP) in any region. Better latency + failover than the public internet.

## Global Accelerator vs CloudFront

| | Global Accelerator | CloudFront |
|---|---|---|
| Protocol | TCP/UDP (any) | HTTP/HTTPS |
| Caching | No | Yes |
| Static IPs | Yes (anycast) | No (DNS-based) |
| Best for | Gaming, IoT, gRPC, non-HTTP, "static IP to whitelist" | Web content, APIs |

If your traffic is HTTP and benefits from caching → CloudFront.
If it's TCP/UDP or you need static IPs → Global Accelerator.

## Key concepts

- **Accelerator** — top-level resource. Get 2 anycast IPv4 addresses.
- **Listener** — port + protocol.
- **Endpoint group** — region + endpoints in that region with weights.
- **Endpoint** — ALB / NLB / EIP / EC2.
- **Health checks** — automatic failover between regions on unhealthy.
- **Traffic dial** — % of traffic per region (for canary).

## Real-world example

> A real-time game with players globally:
> - 4 game-server clusters in `us-east-1`, `eu-west-1`, `ap-southeast-1`, `sa-east-1`.
> - Players hit one of 2 anycast IPs; AWS routes them to the nearest healthy region.
> - On region failure, traffic shifts in seconds.

## Pricing

- **$0.025/hr per accelerator** (~$18/mo).
- **Data processed:** $0.005/GB in DT-Premium fee + standard egress.

## Gotchas

- Doesn't cache — pure routing.
- Endpoints must be public-facing (ALB internet-facing, NLB internet-facing, EIP-bound EC2).
- Two IPs per accelerator; whitelist both.
- Not free — only worth it for the specific use cases.

## Related

- [CloudFront](./cloudfront.md)
- [ALB / NLB](./elb.md)
- [Route 53](./route53.md) — latency-based routing is a cheaper alternative
