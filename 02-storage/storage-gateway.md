# Storage Gateway

**TL;DR** — Appliance you run on-prem that connects local storage to AWS. Three flavors: file, volume, tape.

## What it is

A virtual appliance (VMware/Hyper-V/EC2 AMI/hardware) you install in your data center. On-prem apps talk to it via standard protocols (NFS, SMB, iSCSI, VTL); it lazily moves data to AWS.

## The three flavors

### 1. File Gateway (S3 File Gateway / FSx File Gateway)
- Presents an NFS/SMB share locally.
- Files are stored as objects in S3 (or cached for FSx).
- Use: cloud backup, lift-and-shift file shares, hybrid analytics.

### 2. Volume Gateway
- Presents iSCSI block volumes.
- **Cached mode** — primary data in S3, hot working set on-prem.
- **Stored mode** — primary on-prem, async backup to S3.
- Use: hybrid block storage, async DR.

### 3. Tape Gateway (VTL)
- Pretends to be a tape library (VTL).
- Existing backup software (Veeam, NetBackup, CommVault) writes "tapes" that go to S3 / Glacier.
- Use: replace physical tape libraries.

## Real-world example

> A mid-size bank has old NetBackup workflows writing to physical LTO tapes.
> - Install Tape Gateway appliance in their data center.
> - NetBackup writes virtual tapes → they go to Glacier Deep Archive.
> - Saves on tape hardware + offsite storage; compliance still satisfied.

## Pricing

- **Gateway itself:** free.
- You pay for:
  - S3/Glacier storage for archived data.
  - Data transfer out (if retrieving).
  - The on-prem hardware/VM hosting it.

## Gotchas

- **Initial sync is slow** unless you seed with Snowball.
- **Cache sizing matters.** Too small a cache = cache misses = slow reads.
- **Not a CDN** — it's for backup/archive/hybrid, not perf-critical workloads.

## Related

- [DataSync](../12-migration/datasync.md) — for one-shot bulk migration
- [Snow Family](../12-migration/snow-family.md) — for physical data shipment
- [S3](./s3.md)
- [Glacier](./glacier.md)
