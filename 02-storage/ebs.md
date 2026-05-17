# EBS — Elastic Block Store

**TL;DR** — Virtual disks for EC2. SSD or HDD, attach/detach to instances, snapshot to S3. Lives in one AZ.

## What it is

Block-level storage volumes you attach to EC2 (like a virtual hard drive). The OS sees `/dev/xvdf` or similar. Persistent across instance reboots; survives termination if `DeleteOnTermination=false`.

## Why it exists

EC2 instance store is fast but ephemeral — gone when the instance stops. EBS gives you **persistent**, snapshottable, resizable block storage.

## Volume types

| Type | Use | IOPS | Throughput | $/GB-mo |
|---|---|---|---|---|
| **gp3** (SSD, default) | General purpose | 3000-16k (provisioned) | 125-1000 MB/s | $0.08 |
| gp2 (SSD, old) | Legacy general | scales with size | scales with size | $0.10 |
| **io2 Block Express** | High-IOPS, latency-sensitive DBs | up to 256k | 4000 MB/s | $0.125 + IOPS |
| io2 | High-IOPS | up to 64k | 1000 MB/s | $0.125 + IOPS |
| **st1** (HDD) | Big sequential workloads | low | up to 500 MB/s | $0.045 |
| sc1 (HDD, cold) | Infrequently accessed | very low | up to 250 MB/s | $0.015 |

**`gp3` is the default for almost everything.** ~20% cheaper than `gp2`, same SSD performance.

## Key concepts

- **Volume** — the disk itself, lives in one AZ.
- **Snapshot** — incremental backup stored in S3 (region-wide, not AZ).
- **AMI** — built from one or more EBS snapshots.
- **Encryption** — KMS-encrypted at rest; effectively free, no perf hit.
- **Multi-Attach** (io1/io2) — attach one volume to multiple Nitro instances (clustered FS only).
- **Fast Snapshot Restore** — pre-warm a volume from snapshot for instant full-speed reads.

## Real-world example

> A Postgres instance on EC2:
> - Root volume: 20 GB gp3.
> - Data volume: 200 GB gp3 with 6000 IOPS provisioned (mounted on `/var/lib/postgresql`).
> - Daily lifecycle-managed snapshots, cross-region copy for DR.

## Usage

### Create + attach

```bash
aws ec2 create-volume \
  --availability-zone ap-south-1a \
  --volume-type gp3 --size 100 \
  --iops 5000 --throughput 250 \
  --encrypted \
  --tag-specifications 'ResourceType=volume,Tags=[{Key=Name,Value=data}]'
# → vol-0123...

aws ec2 attach-volume --volume-id vol-0123... --instance-id i-0abc... --device /dev/sdf
```

Inside the instance:
```bash
# Find the device (Nitro renames /dev/sdf -> /dev/nvme1n1)
lsblk
sudo mkfs -t xfs /dev/nvme1n1
sudo mkdir -p /data && sudo mount /dev/nvme1n1 /data
echo "UUID=$(sudo blkid -s UUID -o value /dev/nvme1n1)  /data  xfs  defaults,nofail  0  2" | sudo tee -a /etc/fstab
```

### Resize live

```bash
aws ec2 modify-volume --volume-id vol-0123... --size 200 --iops 8000
# Then grow the partition + filesystem inside the OS
sudo growpart /dev/nvme1n1 1
sudo xfs_growfs /data
```

### Snapshot

```bash
aws ec2 create-snapshot --volume-id vol-0123... --description "nightly"
```

Schedule via **Data Lifecycle Manager** (DLM):
```bash
aws dlm create-lifecycle-policy \
  --execution-role-arn arn:aws:iam::123456789012:role/AWSDataLifecycleManagerDefaultRole \
  --description "Daily snapshots of data volumes" \
  --state ENABLED \
  --policy-details file://dlm-policy.json
```

## Pricing

- **Storage:** per GB-month of *provisioned* size (not used).
- **IOPS (io2/gp3 above baseline):** per IOPS-month.
- **Snapshots:** $0.05/GB-month (incremental — only changed blocks).
- **Snapshot transfer cross-region:** $0.02/GB.

## Gotchas

- **EBS lives in one AZ.** An instance in `1a` can't attach a volume in `1b`. Use a snapshot → create-in-other-AZ.
- **Snapshots are incremental but charges are per used block** — deleting one mid-chain doesn't necessarily free that block.
- **Unused volumes still cost money.** Audit and delete or snapshot.
- **`DeleteOnTermination=false`** on root volume? You'll accumulate orphans.
- **gp2 → gp3 migration is free, faster, cheaper.** Do it.
- **Performance is per-volume.** Striping multiple volumes (RAID 0) for higher aggregate IOPS is a thing — but io2 BX is usually enough.
- **First reads from a snapshot are slow** (lazy load). Use Fast Snapshot Restore for DR drills.

## Related

- [EC2](../01-compute/ec2.md)
- [S3](./s3.md) — for non-block storage
- [EFS](./efs.md) — for shared file system
- [Backup](#) — AWS Backup orchestrates EBS snapshots
