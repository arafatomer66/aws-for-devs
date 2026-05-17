# FSx — Managed File Systems

**TL;DR** — Four managed file systems for specialized workloads: Windows (SMB), Lustre (HPC), NetApp ONTAP (enterprise), OpenZFS (Unix/Linux).

## The four flavors

### 1. FSx for Windows File Server (SMB)
- Native Windows file share via SMB protocol.
- Active Directory integration (AWS Managed AD or your own).
- For: Windows apps, .NET apps, lift-and-shift Windows workloads.

### 2. FSx for Lustre
- Parallel file system. Insanely fast (TB/s throughput possible).
- Tight S3 integration — link a bucket, get a POSIX view.
- For: HPC, ML training, genomics, video rendering.
- Scratch (ephemeral, blazing) vs Persistent (durable) deployment types.

### 3. FSx for NetApp ONTAP
- Full-fledged ONTAP — NFS + SMB + iSCSI, snapshots, dedup, replication.
- For: enterprise lift-and-shift, hybrid cloud (SnapMirror to/from on-prem ONTAP).

### 4. FSx for OpenZFS
- ZFS file system, NFSv3/v4.
- Snapshots, clones, dedup.
- For: Linux/Unix shares with ZFS features.

## Real-world example

> ML team trains models on 50 TB of imagery.
> - Imagery lives in S3.
> - They link an FSx for Lustre `scratch` file system to the S3 bucket.
> - Training EC2 instances mount Lustre → 1 TB/s aggregate reads.
> - When done, they delete the Lustre FS — data stays in S3.

## Pricing snapshot

Roughly:
- Windows: ~$0.13/GB-mo SSD, plus throughput capacity.
- Lustre Scratch: ~$0.14/GB-mo.
- Lustre Persistent: ~$0.20/GB-mo (replicated).
- ONTAP: ~$0.30/GB-mo SSD pool.
- OpenZFS: ~$0.09/GB-mo.

## Quick decision

| Workload | Pick |
|---|---|
| Windows / .NET / SMB clients | FSx for Windows |
| HPC, ML training, video rendering | FSx for Lustre |
| Hybrid w/ existing NetApp on-prem | FSx for ONTAP |
| Linux with ZFS features | FSx for OpenZFS |
| Just need shared POSIX FS on Linux | **EFS** (cheaper, simpler) |

## Gotchas

- **Lustre is fast but specialized** — POSIX-like, not strict POSIX.
- **Windows requires AD** — set this up first.
- **All FSx variants live in one AZ** by default — use Multi-AZ deployment types for HA (extra cost).

## Related

- [EFS](./efs.md)
- [S3](./s3.md)
- [Storage Gateway](./storage-gateway.md)
