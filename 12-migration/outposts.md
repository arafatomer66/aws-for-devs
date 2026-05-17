# Outposts — AWS in Your Data Center

**TL;DR** — Racks (or 1U/2U servers) of AWS hardware shipped to your data center. Run EC2 / EBS / ECS / EKS / RDS / S3 locally with AWS APIs. For latency-bound or regulatory workloads.

## What it is

A physical extension of your AWS Region into your premises. The Outposts rack is managed by AWS (they remote-update it); you provision EC2 / EBS / etc. on it using the same console / CDK / API.

## Form factors

- **Outposts rack** — 42U full rack of compute + storage.
- **Outposts servers** — 1U / 2U units for smaller sites (retail, factories, edge).

## Why use it

- **Low latency** to local equipment (factory floors, hospitals).
- **Data residency** that doesn't allow cloud.
- **Lift-and-shift** apps that can't tolerate WAN latency to a Region.

## Real-world example

> A manufacturing plant has 50 robots requiring <1 ms commands. They put an Outpost rack on-site:
> - EC2 instances run control software locally.
> - Same instances reachable from AWS Region for management.
> - Data backup to S3 in Region via the WAN link.

## Pricing

- 3-year reserved purchase, capacity-based (vCPUs + GiB + GB storage).
- Hardware + AWS service fee.
- Tens of thousands per month, not for everyone.

## Gotchas

- **Local AWS service set is limited** — not every AWS service runs on Outposts.
- **Outpost-to-Region connection** is your responsibility (DX usually).
- **AWS still operates it** — you don't open the rack.

## Related

- [Direct Connect](../04-networking/direct-connect.md)
- [VPC](../04-networking/vpc.md) — Outposts subnets behave like VPC subnets
