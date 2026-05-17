# Direct Connect

**TL;DR** — A dedicated physical fiber from your data center / colo to AWS. Consistent low latency, predictable bandwidth, lower egress cost. For enterprises with serious hybrid needs.

## What it is

A private 1/10/100 Gbps line between your premises (via an AWS Direct Connect location, often a Telehouse / Equinix POP) and AWS. Bypasses the public internet.

## Why it exists

VPN over internet works but:
- Latency varies.
- Bandwidth limited by your internet pipe.
- Egress over internet is $0.09/GB.

DX provides consistent, dedicated capacity with cheaper egress (~$0.02/GB).

## Key concepts

- **Connection** — physical port at a DX location.
- **VIF (Virtual Interface):**
  - **Private VIF** — to a VPC.
  - **Public VIF** — to AWS public services (S3 endpoints, etc.).
  - **Transit VIF** — to a Transit Gateway.
- **LAG (Link Aggregation Group)** — bond multiple physical ports.
- **MACsec** — link-layer encryption.
- **Direct Connect Gateway** — connect a DX to multiple VPCs across regions/accounts.

## Real-world example

> A hospital with 50 TB/day of medical imaging uploads:
> - Dual 10 Gbps DX from their data center to AWS via an Equinix POP.
> - Site-to-site VPN over the DX for IP encryption.
> - Egress out of S3 to clinics is via Public VIF → cheaper than internet.

## Process

DX is **not click-to-provision** in minutes. Steps:
1. Order via AWS Console — get a Letter of Authorization (LOA).
2. Coordinate with your colo provider to terminate a cross-connect.
3. Configure BGP peering.
4. Create VIFs.
5. Test.

Typical time: weeks to months.

## Pricing

- **Port hour:** $0.30/hr 1 Gbps, $2.25/hr 10 Gbps.
- **Egress over DX:** ~$0.02/GB (vs $0.09 internet).
- **Plus your colo/cross-connect fee.**

Only justified at scale or for strict compliance.

## When to choose VPN vs DX

| | Site-to-Site VPN | Direct Connect |
|---|---|---|
| Setup time | Minutes | Weeks |
| Cost | Low | High fixed |
| Latency | Internet, variable | Consistent |
| Bandwidth | Limited by internet | 1-100 Gbps dedicated |
| Encryption | Built-in IPsec | None (or MACsec / VPN over DX) |

Many real architectures use **both**: DX as primary, VPN as backup.

## Gotchas

- **DX is not encrypted by default.** Use IPsec VPN over the DX or MACsec.
- **One DX = single point of failure.** Use two at different POPs for HA.
- **Regional**: DX is to one region; use a DX Gateway to reach others.
- **Long setup time.** Plan months ahead.

## Related

- [VPN Gateway](./vpn.md)
- [Transit Gateway](./transit-gateway.md)
- [VPC](./vpc.md)
