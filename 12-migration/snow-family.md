# Snow Family — Physical Data Transfer

**TL;DR** — AWS ships you ruggedized hardware. You load data, ship it back. For PB-scale transfers, edge compute, or air-gapped environments. Mostly replaced by DataSync for connected sites.

## The devices

- **Snowcone** — 8 lb, 8 TB usable. Edge compute / small transfers.
- **Snowball Edge Storage Optimized** — ~80 TB usable.
- **Snowball Edge Compute Optimized** — fewer TB but EC2 + GPU.
- **Snowmobile** (retired-ish) — a literal shipping container with 100 PB.

## Real-world example

> Production company shoots 800 TB of 8K footage on a remote location with no internet.
> - AWS ships a Snowball Edge.
> - Crew copies media onto it via NFS/S3 API at the site.
> - Ship back to AWS — data ingested to S3 within a week.

## Usage

Order from console → AWS ships the device → you connect to your network → use the AWS OpsHub app (or S3 API) to copy → ship back.

## Pricing

- Service fee per device + per-day rental + shipping.
- Snowball Edge ≈ $300 service fee + ~$30/day after 10 days free.

## Gotchas

- **Physical logistics** — customs, shipping delays.
- **Encryption** — data encrypted with KMS-managed key on device.
- **Edge compute** is limited — useful for pre-ingest transforms, not heavy ML.

## Related

- [DataSync](./datasync.md)
- [Storage Gateway](../02-storage/storage-gateway.md)
