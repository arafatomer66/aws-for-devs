# MGN — Application Migration Service

**TL;DR** — Lift-and-shift servers (VMware / Hyper-V / physical / other clouds) into EC2. Block-level replication, cut over with minutes of downtime.

## What it is

MGN installs a tiny **replication agent** on each source server. The agent continuously replicates disks to a low-cost EC2 staging area in your AWS account. When ready, you "launch a test instance" or "cut over" — MGN spins up your server as an EC2.

## Why it exists

If you want to **rehost** workloads to AWS without re-architecting (a "6 R" strategy: Rehost), MGN gives you a near-zero-downtime migration path. Successor to the older CloudEndure Migration.

## Real-world example

> Company migrating 200 VMs from VMware:
> - Install MGN agent on each VM (Linux or Windows).
> - Disks replicate to AWS over the internet or VPN/DX.
> - Test launches in AWS dev account — verify boots fine.
> - Cut over wave by wave: stop the source VM, launch the AWS instance, switch DNS.

## Usage

Console-driven mostly. Agents installed via:
```bash
# Linux
wget https://aws-application-migration-service-<region>.s3.amazonaws.com/latest/linux/aws-replication-installer-init.py
sudo python3 aws-replication-installer-init.py
```

## Pricing

- **2,160 hours (90 days) free per server.**
- After: $0.042 per hour per replicating server.
- Plus the underlying EC2 + EBS during replication and post-cutover.

## Gotchas

- **Bandwidth from on-prem to AWS** — plan for the working set + delta rates.
- **Cutover takes minutes** for most servers (depends on size + filesystems).
- **Application-level migration** (DBs, AD) often better via other services — MGN works but isn't aware of app semantics.
- **Post-cutover modernization** is your job — MGN doesn't refactor for you.

## Related

- [DMS](./dms.md) — for databases
- [DataSync](./datasync.md) — for file data
