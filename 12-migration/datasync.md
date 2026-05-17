# DataSync

**TL;DR** — Bulk move + sync files between on-prem (NFS/SMB) ↔ AWS storage (S3 / EFS / FSx). Faster, cheaper, more reliable than `rsync` over the internet.

## What it is

A managed data-transfer service. You install a DataSync **agent** VM on-prem (or run in AWS if both source and destination are AWS). Agent talks to the DataSync service over an encrypted tunnel. Optimized for large transfers — checksum, retry, parallelism.

## Source / destination matrix

- **On-prem:** NFS, SMB, HDFS, self-managed object stores.
- **AWS:** S3, EFS, FSx for Windows / Lustre / NetApp / OpenZFS.
- **AWS-to-AWS** (e.g. cross-region S3): supported.
- **Other clouds** (GCS, Azure Blob): supported.

## Real-world example

> Migrate 500 TB of media files from on-prem NAS to S3:
> - Install DataSync agent on a VM in the data center.
> - Source location: NFS share. Destination: S3 bucket.
> - Run task with 80 Gbps bandwidth cap.
> - DataSync handles parallelism + retries; completes in a few days.

## Usage

```bash
# Create agent (returns activation key after on-prem VM phones home)
aws datasync create-agent --activation-key <key> --agent-name onprem-agent

# Locations
aws datasync create-location-nfs --server-hostname 10.0.0.5 --subdirectory /export/media \
  --on-prem-config AgentArns=arn:aws:datasync:..:agent/onprem-agent
aws datasync create-location-s3 --s3-bucket-arn arn:aws:s3:::media-bucket --subdirectory /

# Task
aws datasync create-task --source-location-arn arn:... --destination-location-arn arn:... \
  --options 'VerifyMode=ONLY_FILES_TRANSFERRED,PreserveDeletedFiles=PRESERVE,BytesPerSecond=10000000000'

aws datasync start-task-execution --task-arn arn:...
```

## Pricing

- **$0.0125 per GB transferred.**
- Plus standard storage rates at the destination.

A 500 TB transfer ≈ $6,250 + S3 storage.

## DataSync vs Snow Family vs Storage Gateway

- **DataSync** — online over the network (or DX). Best for ongoing or one-time TB-scale.
- **Snow Family** — physical appliance shipped to you for PB-scale or no-internet.
- **Storage Gateway** — ongoing hybrid (on-prem caches AWS storage).

## Gotchas

- **Bandwidth** — cap the rate so you don't saturate your office.
- **Permissions** — source/destination service-role required.
- **Filters** — include/exclude patterns supported.
- **Checksums** — verify after transfer; adds time but worth it.
- **For S3 destinations** — files map to objects; folder semantics emulated.

## Related

- [Snow Family](./snow-family.md)
- [Storage Gateway](../02-storage/storage-gateway.md)
- [DMS](./dms.md)
