# EC2 — Elastic Compute Cloud

**TL;DR** — Virtual machines you rent by the second. The OG AWS service (launched 2006). If you can SSH into it, it's probably EC2.

## What it is

A "virtual server" running on AWS hardware. You pick:
- **Instance type** — CPU/RAM/network combo (e.g. `t3.medium`, `m6i.large`, `c7g.xlarge`).
- **AMI** — Amazon Machine Image; the OS snapshot to boot from (Amazon Linux 2023, Ubuntu, Windows, your custom image, etc.).
- **Storage** — EBS volume(s) attached as disks.
- **Network** — VPC, subnet, security group.
- **Key pair** — for SSH login.

## Why it exists

Before EC2, you bought physical servers, racked them, paid for cooling. EC2 lets you spin up a server in 60 seconds and shut it down in 60 seconds. **Per-second billing** = no waste.

## Key concepts

- **Instance** — one VM.
- **AMI** — disk image template (your VM boots from this). Region-specific.
- **Instance type** — `<family><gen>.<size>` like `m6i.large`. Family says workload type, gen says hardware generation, size says how big.
- **EBS volume** — the virtual disk. Persists when instance stops.
- **Instance store** — local NVMe SSD on the host. **Ephemeral** — wiped on stop.
- **Security group (SG)** — stateful firewall around the instance.
- **Key pair** — SSH public/private key. Lose the .pem, lose access.
- **Elastic IP (EIP)** — static public IPv4. Free while attached, $0.005/hr unattached.
- **Auto Scaling Group (ASG)** — automatically maintains N instances, scales up/down.
- **Launch Template** — recipe for booting new instances (used by ASGs).
- **Placement Group** — control how instances are spread (cluster, spread, partition).
- **User data** — script that runs at first boot (cloud-init).

## Instance type families

- **`t`** (Burstable, t3/t4g) — small, cheap, CPU credits. Default for low/spiky load.
- **`m`** (General purpose, m6i/m7g) — balanced CPU/RAM. Default for web servers.
- **`c`** (Compute-optimized, c7i/c7g) — high CPU, lower RAM. For batch / CPU work.
- **`r`** (Memory-optimized, r7i) — lots of RAM. For caches, in-memory DBs.
- **`x`** (Extreme memory, x2idn) — TBs of RAM for SAP HANA etc.
- **`i`** (Storage-optimized) — local NVMe SSD. For databases.
- **`g` / `p`** (GPU) — ML training, gaming, rendering.
- **`inf` / `trn`** — AWS custom ML accelerators (Inferentia, Trainium).
- **Graviton (`g` suffix on size: `m6g`, `c7g`)** — ARM processors, ~20% cheaper, faster perf-per-watt.

## Pricing models (compute)

- **On-Demand** — pay per second, no commitment.
- **Spot** — up to 90% off, can be reclaimed in 2 mins.
- **Savings Plans** — commit $/hr for 1–3 yrs, save 30–72%.
- **Reserved Instances** — older, similar to Savings Plans.
- **Dedicated Host** — entire physical server, for BYOL licensing.
- **Dedicated Instance** — runs on hardware dedicated to your account.

## Real-world example

> ShareDeal's order-processing background workers run as Spot instances in an Auto Scaling Group. When AWS gives a 2-minute Spot termination warning, the worker finishes the current job and refuses new ones; the ASG launches a replacement automatically.

## Usage — launching an instance

### Via CLI

```bash
# Get the latest Amazon Linux 2023 AMI in your region
AMI=$(aws ec2 describe-images \
  --owners amazon \
  --filters "Name=name,Values=al2023-ami-2023.*-x86_64" \
  --query 'Images | sort_by(@,&CreationDate) | [-1].ImageId' \
  --output text)

aws ec2 run-instances \
  --image-id "$AMI" \
  --instance-type t3.micro \
  --key-name my-keypair \
  --security-group-ids sg-0123456789abcdef0 \
  --subnet-id subnet-0123456789abcdef0 \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=web-1},{Key=Env,Value=dev}]' \
  --user-data file://bootstrap.sh
```

### Via Terraform

```hcl
resource "aws_instance" "web" {
  ami           = "ami-0abcdef1234567890"
  instance_type = "t3.micro"
  subnet_id     = aws_subnet.public.id
  vpc_security_group_ids = [aws_security_group.web.id]
  user_data = file("bootstrap.sh")
  tags = { Name = "web-1", Env = "dev" }
}
```

### Via CDK (TypeScript)

```ts
new ec2.Instance(this, 'Web', {
  vpc,
  instanceType: ec2.InstanceType.of(ec2.InstanceClass.T3, ec2.InstanceSize.MICRO),
  machineImage: ec2.MachineImage.latestAmazonLinux2023(),
  securityGroup: webSg,
});
```

## SSH in

```bash
chmod 400 my-keypair.pem
ssh -i my-keypair.pem ec2-user@<public-ip>     # Amazon Linux
ssh -i my-keypair.pem ubuntu@<public-ip>       # Ubuntu
```

Or better: use **Session Manager** (no SSH port open, IAM-auth'd):
```bash
aws ssm start-session --target i-0abcdef1234567890
```

## Bootstrapping with user data

```bash
#!/bin/bash
dnf update -y
dnf install -y nginx
systemctl enable --now nginx
echo "<h1>Hello from $(hostname)</h1>" > /usr/share/nginx/html/index.html
```

Pass this script as `--user-data file://bootstrap.sh`. Cloud-init runs it on first boot.

## Instance states

`pending → running → stopping → stopped → terminated`

- **Stopped** — no compute charge, but you still pay for EBS volume.
- **Terminated** — gone. Root EBS volume is deleted by default.

## Auto Scaling

Don't run one EC2 in production. Use an **Auto Scaling Group**:

```bash
aws autoscaling create-auto-scaling-group \
  --auto-scaling-group-name web-asg \
  --launch-template LaunchTemplateName=web-lt \
  --min-size 2 --max-size 10 --desired-capacity 3 \
  --vpc-zone-identifier "subnet-a,subnet-b,subnet-c" \
  --target-group-arns arn:aws:elasticloadbalancing:...:targetgroup/web/...
```

Scale on CPU > 70%, request count, or custom CloudWatch metrics.

## Gotchas

- **Forgetting instances costs money.** A running `m5.large` ≈ $70/mo. Tag everything and use Budgets.
- **`stop` vs `terminate`** — terminate is permanent; data on the root volume is gone.
- **Instance store ≠ EBS.** Instance store is wiped on stop. EBS persists.
- **AMIs are regional.** Copy to another region if you want to launch there.
- **Security groups are stateful** — if you allow inbound on 80, reply traffic out is automatic.
- **`t.` burstable instances** can run out of CPU credits and throttle badly under sustained load.
- **Public IPs from default pool will rotate** if you stop+start. Use an EIP or a load balancer for stability.
- **Graviton (`g` family) is ARM** — recompile your container images for `arm64`.

## Related

- [EBS](../02-storage/ebs.md) — disks for EC2
- [VPC](../04-networking/vpc.md) — where EC2 lives
- [Auto Scaling](#auto-scaling)
- [Fargate](./fargate.md) — same thing but for containers without managing EC2
- [Lambda](./lambda.md) — alternative if you don't need a long-running server
