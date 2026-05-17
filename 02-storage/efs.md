# EFS — Elastic File System

**TL;DR** — Managed NFS. Mount the same file system on many EC2 / Fargate / Lambda concurrently. Scales automatically. Pay per GB stored.

## What it is

A POSIX file system you mount over NFSv4.1. Unlike EBS (one volume → one instance), EFS supports **thousands of clients reading/writing the same files at once**. Lives across AZs.

## Why it exists

When you need:
- A shared filesystem across many EC2/containers/Lambdas.
- Auto-growing storage (TB → PB).
- Linux POSIX semantics (file locks, permissions, etc.).

## Key concepts

- **File system** — the top-level resource, regional, spans AZs.
- **Mount target** — an ENI in a subnet that clients mount via NFS.
- **Performance modes:**
  - **General Purpose** (default) — lowest latency.
  - **Max I/O** — higher throughput, slightly higher latency (legacy; not recommended now).
- **Throughput modes:**
  - **Elastic** (default, 2023+) — auto-scales, pay per use.
  - **Provisioned** — set fixed MB/s.
  - **Bursting** — older default.
- **Lifecycle policy** — auto-move infrequently-accessed files to IA / Archive tiers.
- **Access points** — POSIX user + path prefix, lets you give different IAM principals different views.
- **Mount via DNS** — `fs-xxxx.efs.<region>.amazonaws.com`.

## Real-world example

> A Fargate web cluster runs Drupal across 6 tasks. They all share `/var/www/html` via EFS — uploaded user files are visible to all instances without sticky sessions.

## Usage

### Create + mount

```bash
aws efs create-file-system \
  --creation-token web-content \
  --performance-mode generalPurpose \
  --throughput-mode elastic \
  --encrypted \
  --tags Key=Name,Value=web-content

# Create mount targets in each AZ subnet
aws efs create-mount-target --file-system-id fs-0123... --subnet-id subnet-a --security-groups sg-efs
aws efs create-mount-target --file-system-id fs-0123... --subnet-id subnet-b --security-groups sg-efs
```

On an Amazon Linux EC2:
```bash
sudo dnf install -y amazon-efs-utils
sudo mkdir -p /mnt/efs
sudo mount -t efs fs-0123...:/ /mnt/efs
# Persistent
echo "fs-0123...:/  /mnt/efs  efs  _netdev,tls  0 0" | sudo tee -a /etc/fstab
```

### Fargate (task definition)

```json
"volumes": [{
  "name": "shared",
  "efsVolumeConfiguration": {
    "fileSystemId": "fs-0123...",
    "transitEncryption": "ENABLED",
    "authorizationConfig": { "accessPointId": "fsap-0123..." }
  }
}],
"containerDefinitions": [{
  "name": "web",
  "mountPoints": [{ "sourceVolume": "shared", "containerPath": "/var/www/html" }],
  ...
}]
```

### Lambda

Attach EFS to Lambda for shared ML model files, caches, etc. (one EFS access point per Lambda).

## Pricing

- **Standard storage:** ~$0.30/GB-mo (3× S3 Standard cost).
- **Infrequent Access (IA):** ~$0.025/GB-mo + retrieval fees.
- **Archive:** ~$0.008/GB-mo + larger retrieval fee.
- **Lifecycle policy** auto-moves data — usually pays for itself fast.
- **Elastic throughput:** ~$0.03/GB read, ~$0.06/GB write.

## When EFS vs S3 vs EBS

| Need | Pick |
|---|---|
| POSIX file system, shared by many clients | **EFS** |
| Block device for one EC2 (DB, OS) | **EBS** |
| Object store / static files / data lake | **S3** |
| Windows file share | **FSx for Windows** |
| High-perf HPC | **FSx for Lustre** |

## Gotchas

- **NFS port 2049** open in security group required.
- **Cold reads after IA transition** add ~10 ms latency. Don't transition active hot files.
- **POSIX users matter.** Container user UID must match file ownership, or use access points to enforce a UID.
- **`amazon-efs-utils`** handles TLS + retries; raw `mount -t nfs4` works but skips niceties.
- **More expensive than EBS** for the same GB — don't use EFS as a single-instance disk.
- **Latency is higher than EBS** (NFS over network). Not great for low-latency DBs.

## Related

- [EBS](./ebs.md)
- [FSx](./fsx.md)
- [S3](./s3.md)
