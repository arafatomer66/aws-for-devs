# Elastic Load Balancing

**TL;DR** — Managed load balancers. Three flavors: ALB (HTTP/HTTPS), NLB (TCP/UDP, ultra fast), GLB (network appliances). Plus the legacy CLB.

## The four (active) types

| | ALB | NLB | GLB | CLB (legacy) |
|---|---|---|---|---|
| Layer | 7 (HTTP/HTTPS/WS) | 4 (TCP/UDP/TLS) | 3 (IP) | 4/7 |
| Use | Web apps, APIs | Low-latency, high-throughput TCP, static IPs | Insert firewalls/IDS appliances | Don't use new |
| Routing | Host, path, headers, query, source IP | Listener port | All traffic to appliance | Basic |
| Targets | EC2, ECS, Lambda, IPs | EC2, IPs (ECS, on-prem) | Network appliances | EC2 |
| Static IP | No (auto DNS) | Yes (1 per AZ) | No | No |
| WebSockets | Yes | Yes (just TCP) | N/A | Limited |

## Application Load Balancer (ALB) — the default

For HTTP/HTTPS apps. Path-based routing, host-based routing, header-based routing, redirects, fixed responses, OIDC auth, WebSockets, HTTP/2.

### Key concepts

- **Listener** — port + protocol (e.g. 443/HTTPS).
- **Rule** — matched conditions (host, path, header) → action (forward to target group, redirect, fixed response).
- **Target group** — set of targets (EC2 instances, ECS tasks, IPs, Lambda). Health-checked.
- **Sticky sessions** — cookie-based.
- **Connection draining** — graceful target removal.

### Real-world example

> ShareDeal:
> - ALB on 443, ACM cert for `*.selefe.com`.
> - Rule: `host = api.selefe.com, path /api/*` → API target group (Fargate tasks).
> - Rule: `host = admin.selefe.com` → admin target group.
> - Default action: redirect to `https://selefe.com`.

### Usage (CDK)

```ts
const alb = new elbv2.ApplicationLoadBalancer(this, "Alb", { vpc, internetFacing: true });
const listener = alb.addListener("HttpsListener", { port: 443, certificates: [cert] });

const apiTg = new elbv2.ApplicationTargetGroup(this, "ApiTg", {
  vpc, port: 8080, protocol: elbv2.ApplicationProtocol.HTTP,
  healthCheck: { path: "/health" },
});

listener.addAction("ApiRoute", {
  priority: 10,
  conditions: [elbv2.ListenerCondition.hostHeaders(["api.selefe.com"])],
  action: elbv2.ListenerAction.forward([apiTg]),
});
```

## Network Load Balancer (NLB)

For TCP/UDP. Microsecond-low latency, millions of requests/sec, static IPs (one Elastic IP per AZ), TLS termination optional.

Use NLB when:
- gRPC at scale.
- Game servers (UDP).
- You need static IPs to whitelist.
- Extreme throughput (NLB is faster than ALB).

```ts
const nlb = new elbv2.NetworkLoadBalancer(this, "Nlb", { vpc, internetFacing: true });
nlb.addListener("Tcp", { port: 443, protocol: elbv2.Protocol.TLS, certificates: [cert] })
   .addTargets("Backend", { port: 8080, targets: [ecsService] });
```

## Gateway Load Balancer (GLB)

For deploying network virtual appliances (firewalls, IDS, packet inspection) transparently. You probably won't use this directly unless you build SecOps.

## Pricing

- **ALB:** $0.0225/hr + $0.008 per LCU-hr. ~$16-25/mo idle.
- **NLB:** $0.0225/hr + $0.006 per NLCU-hr. Similar idle.
- **GLB:** $0.0125/hr + $0.004 per GLCU-hr.

LCU = bundle of new connections + active connections + bandwidth + rule evals. Usually a few LCUs/mo for small apps.

## Health checks

- **ALB:** HTTP/HTTPS check, configurable path, codes, interval.
- **NLB:** TCP/HTTP check.
- Configure **healthy threshold** + **unhealthy threshold** + **interval**.
- Use a dedicated `/health` endpoint on your service; don't probe `/` which may exercise DB.

## Gotchas

- **ALB → Lambda** has a 1 MB request limit and 30s timeout (Lambda-imposed).
- **Sticky sessions** make scaling/rolling deploys harder. Avoid if possible.
- **Cross-zone load balancing** — ALB on by default (free), NLB off by default (extra cost when on). Turn on for even spread.
- **Pre-warming** — for huge launches, ask AWS to pre-warm a new ALB; otherwise it ramps over minutes.
- **Internal vs internet-facing** — choose at create, can't change.
- **Public IPv4 costs $0.005/hr each** as of 2024 — affects NLB static IPs slightly.

## Related

- [Route 53](./route53.md) — point DNS at ELB
- [Global Accelerator](./global-accelerator.md)
- [API Gateway](./api-gateway.md) — sometimes a better fit than ALB
- [CloudFront](./cloudfront.md) — in front of ALB for cache + WAF
