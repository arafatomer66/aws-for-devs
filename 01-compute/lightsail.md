# Lightsail

**TL;DR** — Simple VPS with flat pricing. Pre-packaged stacks (WordPress, LAMP, Node, Django). For when EC2 feels like overkill.

## What it is

A "DigitalOcean-style" simple cloud product baked on top of EC2. You pick a plan ($3.50–$160/mo), get an instance + bandwidth + static IP, and click-deploy popular stacks.

## Why it exists

EC2's pricing has ~50 dimensions (instance + EBS + bandwidth + EIPs + …). For students, hobbyists, or simple WordPress sites, that's overwhelming. Lightsail = predictable monthly bill.

## Key concepts

- **Instance** — Lightsail's VM (under the hood: an EC2 with a bundle).
- **Plan** — bundle of CPU/RAM/SSD/transfer at a fixed monthly price.
- **Bundle / Blueprint** — OS image or pre-installed app (WordPress, Node.js, Plesk, etc.).
- **Static IP** — free (unlike EC2's EIPs).
- **Lightsail Containers** — flat-priced container service.
- **Lightsail Databases** — flat-priced MySQL/PostgreSQL.
- **Lightsail Load Balancer** — $18/mo flat.

## Pricing (representative, Linux)

| Plan | RAM | vCPU | SSD | Transfer | Price |
|---|---|---|---|---|---|
| Nano | 512 MB | 2 | 20 GB | 1 TB | $3.50 |
| Micro | 1 GB | 2 | 40 GB | 2 TB | $5 |
| Small | 2 GB | 2 | 60 GB | 3 TB | $10 |
| Medium | 4 GB | 2 | 80 GB | 4 TB | $20 |
| Large | 8 GB | 2 | 160 GB | 5 TB | $40 |
| XL | 16 GB | 4 | 320 GB | 6 TB | $80 |
| 2XL | 32 GB | 8 | 640 GB | 7 TB | $160 |

Windows ~$8 more per tier.

## Real-world example

> A freelancer hosts 8 client WordPress sites on a $20/mo Lightsail with the WordPress Multisite blueprint. Backups daily. Total ops time: maybe 1 hour/month.

## Usage

Mostly via the web console. CLI:

```bash
aws lightsail create-instances \
  --instance-names blog-01 \
  --availability-zone ap-south-1a \
  --blueprint-id wordpress \
  --bundle-id small_3_0 \
  --region ap-south-1
```

Or upgrade to EC2 later:
```bash
aws lightsail export-snapshot --source-snapshot-name blog-01-snap
# Then create an EC2 from the resulting AMI
```

## When NOT to use Lightsail

- You need IAM roles on the instance (Lightsail's IAM integration is limited).
- You need VPC connectivity to other AWS resources (use VPC peering, or just use EC2).
- You need to scale beyond ~16 vCPU/64 GB.
- You need Auto Scaling.

## Gotchas

- **Bandwidth overage** is billed per GB — cheap but not free.
- **Limited region availability** — fewer regions than EC2.
- **No tight IAM integration** out of the box.
- **You can graduate to EC2** by exporting a snapshot, but apps may need re-config.

## Related

- [EC2](./ec2.md) — when you outgrow Lightsail
- [Elastic Beanstalk](./elastic-beanstalk.md)
- [App Runner](./app-runner.md)
