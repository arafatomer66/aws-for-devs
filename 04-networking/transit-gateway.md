# Transit Gateway (TGW)

**TL;DR** — A network hub that connects many VPCs, on-prem networks, and even other TGWs across regions. Replaces messy VPC-peering meshes.

## Why it exists

VPC peering is point-to-point and non-transitive. With 10 VPCs you'd need ~45 peerings. TGW is hub-and-spoke — each VPC attaches once.

## Key concepts

- **TGW** — the hub.
- **Attachment** — connection from a VPC, VPN, DX (via Transit VIF), or another TGW.
- **Route table** — TGW has its own; controls which attachments can reach which.
- **Inter-region peering** — TGWs in different regions can be peered.
- **Multicast** — TGW supports multicast (rare).
- **Network Manager** — single pane of glass for global networks.

## Real-world example

> Holding company with 12 subsidiary AWS accounts:
> - Each VPC attaches to a central TGW (shared via Resource Access Manager).
> - On-prem DC connects via Direct Connect + Transit VIF.
> - Route tables: prod VPCs see each other, dev VPCs are isolated from prod.

## Pricing

- **Attachment-hour:** $0.05/hr per attachment.
- **Data processed:** $0.02/GB through the TGW.

10 VPCs attached + 100 GB/mo = ~$365 + $2 = $367/mo. Pricey for small setups.

## Usage (CLI)

```bash
aws ec2 create-transit-gateway --description "main hub"
aws ec2 create-transit-gateway-vpc-attachment \
  --transit-gateway-id tgw-xxx --vpc-id vpc-xxx \
  --subnet-ids subnet-a subnet-b
```

## TGW vs VPC Peering vs Cloud WAN

- **VPC peering** — simple, free, fine for 2-5 VPCs.
- **TGW** — for many VPCs, hybrid networks.
- **Cloud WAN** — global wide-area network with policy-based segments (TGW-on-steroids, multi-region).

## Gotchas

- **Cross-region TGW peering** has its own data charge.
- **Route tables can be complex** — design segmentation up front.
- **MTU is 8500 within VPC**, 1500 over VPN, 9001 over DX with jumbo enabled. Path MTU varies.
- **Not free.** A 50-VPC org pays real money for TGW.

## Related

- [VPC](./vpc.md)
- [Direct Connect](./direct-connect.md)
- [VPN](./vpn.md)
