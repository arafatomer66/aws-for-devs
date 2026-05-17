# Cloud Map

**TL;DR** — Service discovery. Register service instances; clients look them up by DNS or API. Used by ECS Service Connect / Service Discovery. Like Consul, lighter weight.

## What it is

A managed registry of named services and their endpoints. Two flavors:
- **DNS namespace** — register services as DNS records (queryable via standard DNS).
- **HTTP namespace** — registry-only, queryable via API (not DNS).

Most commonly used **behind the scenes by ECS** — when you enable Service Discovery on an ECS service, it creates Cloud Map records automatically.

## Key concepts

- **Namespace** — top-level scope (`internal.selefe.local`).
- **Service** — a named service (`payments`, `inventory`).
- **Instance** — concrete endpoint (`10.0.1.5:8080`, with health status).
- **Health checks** — DNS-based or Route 53 health checks.

## Real-world example

> ECS Fargate microservices:
> - Cloud Map namespace `selefe.local` (private DNS in VPC).
> - Services: `payments`, `inventory`, `orders`.
> - Each ECS service registers its tasks automatically.
> - From `orders` task, call `payments.selefe.local:8080` — DNS resolves to a healthy task IP.

Without Cloud Map you'd put each service behind an internal ALB. Cloud Map is cheaper for small east-west traffic.

## Usage

### Create a private namespace

```bash
aws servicediscovery create-private-dns-namespace \
  --name selefe.local --vpc vpc-xxxxx
```

### Create a service

```bash
aws servicediscovery create-service --name payments \
  --namespace-id ns-xxx \
  --dns-config 'NamespaceId=ns-xxx,DnsRecords=[{Type=A,TTL=10}]' \
  --health-check-custom-config FailureThreshold=1
```

### Wire into ECS service (Service Discovery)

Easiest: ECS service config with `serviceRegistries`:
```json
"serviceRegistries": [{
  "registryArn": "arn:aws:servicediscovery:..:service/srv-xxx"
}]
```

ECS auto-registers/deregisters task IPs as tasks start/stop.

## ECS Service Connect (newer alternative)

ECS Service Connect builds on Cloud Map + adds Envoy proxy for retries, TLS, traffic shaping. For new ECS architectures, prefer Service Connect over raw Service Discovery.

## Pricing

- **$0.10 per service per month** registered.
- **$0.40 per million DNS queries** (or API discovery requests).

Cheap.

## Gotchas

- **DNS TTL** — set short (10s) for fast failover, but more queries.
- **Health checks** add cost — use `FailureThreshold=1` for fast removal of unhealthy tasks.
- **Public namespaces** also supported, but you usually want private + ALB for north-south.
- **Cross-VPC discovery** needs DNS resolver tricks or peering.

## Related

- [ECS](../01-compute/ecs.md) — Service Connect / Service Discovery
- [Route 53](./route53.md) — Cloud Map uses Route 53 hosted zones under the hood
- [App Mesh](#) — deeper service mesh (mostly being deprecated)
