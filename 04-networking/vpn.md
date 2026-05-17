# VPN — Site-to-Site & Client VPN

**TL;DR** — IPsec tunnels into your VPCs. Two flavors: Site-to-Site (network-to-network) and Client VPN (laptop-to-AWS).

## Site-to-Site VPN

Connect your on-prem network to a VPC over IPsec.

### Components

- **Customer Gateway (CGW)** — represents your on-prem device.
- **Virtual Private Gateway (VGW)** or **Transit Gateway** — AWS side.
- **VPN Connection** — IPsec tunnels (2 per connection for HA).
- BGP (dynamic) or static routing.

### Pricing

- $0.05/hr per VPN connection (~$36/mo).
- Plus data egress.

### Real-world example

> 5-person startup needs to access on-prem ERP from AWS:
> - Set up Site-to-Site VPN from their office firewall (Fortigate) to a VGW on their VPC.
> - On-prem subnet `192.168.10.0/24` reachable from EC2.

## Client VPN

Lets users (laptops) VPN into a VPC. Authenticates via mutual cert, AD, or SAML.

### Components

- **Client VPN endpoint** — server side.
- **Authentication** — certs, AD, federated (SAML).
- **Authorization rules** — which subnets each user/group can reach.
- **OpenVPN protocol** under the hood.

### Pricing

- $0.10/hr per active connection.
- $0.05/hr per associated subnet (HA = multiple).

A 50-user team running 8h/day ≈ $1k/mo. Often a cheaper SaaS VPN works better.

## Usage

### Site-to-Site (rough)

```bash
aws ec2 create-customer-gateway --type ipsec.1 --public-ip 1.2.3.4 --bgp-asn 65000
aws ec2 create-vpn-gateway --type ipsec.1
aws ec2 attach-vpn-gateway --vpc-id vpc-xxx --vpn-gateway-id vgw-xxx
aws ec2 create-vpn-connection --type ipsec.1 \
  --customer-gateway-id cgw-xxx --vpn-gateway-id vgw-xxx --options "StaticRoutesOnly=true"
```

AWS gives you a config file for your on-prem device.

### Client VPN

Console-driven mostly. Generate a client config file, distribute, users connect via the OpenVPN client.

## Gotchas

- **Two tunnels per S2S** — your device must support both for HA.
- **Client VPN is pricier than alternatives** (Tailscale, Twingate, Zscaler) for many use cases.
- **VPN ≠ low latency.** Use Direct Connect for big bandwidth or jittery workloads.
- **MTU/MSS issues** are common — clamp MSS to ~1379 if seeing weird hangs.

## Related

- [Direct Connect](./direct-connect.md)
- [Transit Gateway](./transit-gateway.md)
