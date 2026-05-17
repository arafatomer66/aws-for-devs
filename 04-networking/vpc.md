# VPC — Virtual Private Cloud

**TL;DR** — Your own isolated virtual network on AWS. Subnets, route tables, gateways, security groups. **Almost every AWS resource lives inside a VPC.**

## What it is

A logically isolated network in AWS. You pick a CIDR (e.g. `10.0.0.0/16`), split into subnets across AZs, decide what's public and what's private, and AWS handles the underlying SDN.

## Why it exists

Pre-VPC (EC2-Classic, ancient), every EC2 was on a shared flat network — terrifying. VPC lets you draw boundaries: prod vs dev, public web tier vs private DB tier.

## Key concepts

- **CIDR block** — the IP range, e.g. `10.0.0.0/16` (65,536 addresses). Can't change after create; can add secondary CIDRs.
- **Subnet** — a slice of the VPC in one AZ, e.g. `10.0.1.0/24` in `ap-south-1a`.
  - **Public** — has a route to an Internet Gateway. EC2s here can have public IPs.
  - **Private** — no internet route, or only outbound via NAT.
- **Route Table** — rules for "traffic to X goes via Y." Attached to subnets.
- **Internet Gateway (IGW)** — VPC's connection to the public internet.
- **NAT Gateway** — lets private subnets reach internet (e.g. for `apt-get`, ECR pulls), but blocks inbound.
- **Security Group (SG)** — stateful firewall around an ENI (instance, RDS, Lambda, etc.).
- **Network ACL (NACL)** — stateless subnet-level firewall (less common to customize).
- **Elastic Network Interface (ENI)** — virtual NIC; what gets attached to instances/Lambdas/tasks.
- **VPC Peering** — connect two VPCs. Non-transitive.
- **Transit Gateway** — hub-and-spoke for many VPCs + on-prem.
- **VPC Endpoint** — private connection to AWS services without going through internet.
  - **Gateway endpoint** — for S3 and DynamoDB. Free.
  - **Interface endpoint** — for everything else, via PrivateLink. ~$7/mo per AZ + data.
- **VPC Flow Logs** — log accepted/denied traffic to CloudWatch / S3.

## A canonical VPC layout

```
VPC 10.0.0.0/16  (ap-south-1)
├── 10.0.0.0/24   public-1a       (IGW route)
├── 10.0.1.0/24   public-1b       (IGW route)
├── 10.0.10.0/24  private-app-1a  (NAT route)
├── 10.0.11.0/24  private-app-1b  (NAT route)
├── 10.0.20.0/24  private-db-1a   (no internet)
└── 10.0.21.0/24  private-db-1b   (no internet)
```

Two public subnets for ALB / NAT, two private app subnets for ECS/EC2/Lambda, two private DB subnets for RDS — across two AZs.

## Real-world example

> ShareDeal VPC:
> - `10.0.0.0/16`, 3 AZs.
> - ALB in public subnets, gets a public IP.
> - ECS Fargate tasks in private subnets, talk to ALB internally.
> - Aurora in db subnets, only reachable from app SG.
> - S3 via gateway endpoint — no NAT cost for image uploads.
> - NAT Gateway in one AZ (cost optimization; could be one per AZ for HA).

## Usage

### Quickest: use the default VPC

Every region has a default VPC with `172.31.0.0/16` and one public subnet per AZ. Fine for experiments. **Don't use it for prod** — too permissive.

### Create a real VPC (CDK)

```ts
import { Vpc, SubnetType } from "aws-cdk-lib/aws-ec2";

const vpc = new Vpc(this, "Vpc", {
  ipAddresses: ec2.IpAddresses.cidr("10.0.0.0/16"),
  maxAzs: 3,
  natGateways: 1,  // cost-saver; use 3 in real prod
  subnetConfiguration: [
    { name: "public",      subnetType: SubnetType.PUBLIC,             cidrMask: 24 },
    { name: "private-app", subnetType: SubnetType.PRIVATE_WITH_EGRESS, cidrMask: 24 },
    { name: "private-db",  subnetType: SubnetType.PRIVATE_ISOLATED,    cidrMask: 24 },
  ],
});
```

That's the right default.

### Create a VPC (CLI manually — ugly but instructive)

```bash
aws ec2 create-vpc --cidr-block 10.0.0.0/16
aws ec2 create-subnet --vpc-id vpc-xxx --cidr-block 10.0.0.0/24 --availability-zone ap-south-1a
aws ec2 create-internet-gateway
aws ec2 attach-internet-gateway --internet-gateway-id igw-xxx --vpc-id vpc-xxx
# ... route tables, NAT GW, security groups, etc.
```

Don't do this by hand. Use CDK / Terraform.

## Security groups

Stateful (return traffic auto-allowed).

```bash
aws ec2 create-security-group --group-name web-sg --description "web tier" --vpc-id vpc-xxx
aws ec2 authorize-security-group-ingress --group-id sg-xxx \
  --protocol tcp --port 443 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-ingress --group-id sg-db \
  --protocol tcp --port 5432 --source-group sg-app   # only app SG can reach DB
```

**Reference SGs by ID, not by IP.** Cleaner, lets resources move IPs.

## Subnet IP planning

For a `10.0.0.0/16`:
- `/24` subnets give 251 usable IPs each (AWS reserves 5).
- Plan for elastic growth — too small = ENI exhaustion (especially for ECS awsvpc + EKS pods).
- For EKS, use larger subnets or prefix delegation.

## VPC Endpoints — save on NAT cost

If your Lambdas/EC2 talk to S3, DynamoDB, Secrets Manager, ECR, etc., consider:

- **Gateway endpoint for S3:** free, just a route. Add one per VPC.
- **Gateway endpoint for DynamoDB:** also free.
- **Interface endpoints (PrivateLink):** for STS, SQS, SNS, KMS, SSM, ECR, CloudWatch, etc. ~$7/AZ/mo + data.

Without endpoints, private-subnet traffic to S3 goes via NAT → $$$. Endpoints are usually a net win.

## NAT Gateway gotcha

- **~$32/mo per NAT** standby + **$0.045/GB processed**.
- For a 3-AZ HA NAT setup: $96+/mo idle, before any traffic.
- For staging/dev: use 1 NAT in 1 AZ, accept that an AZ outage breaks outbound from other AZs.
- Or use **NAT instance** on a cheap EC2 (manual, not HA).

## Gotchas

- **CIDR can't be changed** after VPC create. Pick something that won't clash with on-prem or peers (avoid `10.0.0.0/16` if you'll connect to another AWS account that also uses it).
- **Default VPC quirks** — every region has one; some old apps assume its existence.
- **Subnet IPs are scarce** for high-density workloads (EKS pods, Lambda VPC-attached at scale). Plan early.
- **Security groups have rule limits** (default 60 inbound + 60 outbound). Hit the limit on big architectures.
- **NACLs are stateless** — return traffic needs an explicit rule. Avoid customizing them unless you must.
- **Cross-AZ data transfer is $0.01/GB each way** — keep chatty services within an AZ if possible.
- **VPC Flow Logs** are cheap insurance — turn on at the VPC level.

## Related

- [Route 53](./route53.md)
- [Transit Gateway](./transit-gateway.md)
- [PrivateLink](./privatelink.md)
- [ELB](./elb.md)
