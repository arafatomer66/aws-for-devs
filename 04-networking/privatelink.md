# PrivateLink / VPC Endpoints

**TL;DR** — Access AWS services (and SaaS) privately, without going over the public internet. Just an ENI in your VPC with a fancy DNS name.

## What it is

PrivateLink exposes a service via an ENI in your VPC. From your apps it looks like a normal private IP. Traffic never traverses the public internet, doesn't need a NAT.

## Two endpoint types

- **Gateway endpoint** — only for **S3 and DynamoDB**. Free. Just a route table entry.
- **Interface endpoint (PrivateLink)** — for almost everything else (STS, SQS, SNS, KMS, SSM, ECR, Secrets Manager, CloudWatch, Step Functions, Lambda, etc.). ENI in your subnet. Costs ~$7/AZ/mo + data.

## Why it exists

Without endpoints, code in a private subnet calling `s3:GetObject` exits via NAT → internet → S3 (in same region). That's:
1. Expensive (NAT data charge).
2. Sometimes blocked (no NAT? no internet).
3. A security exposure (traffic on the public internet).

Endpoints fix all three.

## Real-world example

> A Lambda in a private subnet reads from S3, writes to DynamoDB, fetches secrets from Secrets Manager.
> - Add S3 + DynamoDB **gateway endpoints** (free).
> - Add **interface endpoint** for Secrets Manager.
> - Result: no NAT needed for these calls. Lower cost, faster, more secure.

## Usage

### Gateway endpoint (S3)

```bash
aws ec2 create-vpc-endpoint --vpc-id vpc-xxx \
  --service-name com.amazonaws.ap-south-1.s3 \
  --route-table-ids rtb-private-1 rtb-private-2
```

### Interface endpoint (Secrets Manager)

```bash
aws ec2 create-vpc-endpoint --vpc-id vpc-xxx \
  --service-name com.amazonaws.ap-south-1.secretsmanager \
  --vpc-endpoint-type Interface \
  --subnet-ids subnet-a subnet-b subnet-c \
  --security-group-ids sg-vpce \
  --private-dns-enabled
```

With private DNS enabled, `secretsmanager.ap-south-1.amazonaws.com` automatically resolves to the endpoint ENI from inside the VPC.

## PrivateLink for third parties

You can also expose your own service (NLB-fronted) to other AWS accounts via PrivateLink. Many SaaS vendors (Snowflake, Datadog, MongoDB Atlas) offer PrivateLink endpoints — no internet exposure for their integration.

## Pricing

- Gateway endpoint: free.
- Interface endpoint: ~$0.01/hr per AZ + $0.01/GB.
- A typical service with 3-AZ HA = ~$22/mo before data.

## Gotchas

- **Endpoints are regional.** Endpoint in `us-east-1` can't reach service in `eu-west-1`.
- **Not all services support gateway endpoints** — only S3 and DDB. Everything else is interface.
- **DNS:** Private DNS overrides public DNS inside the VPC — usually you want this; turn off if you specifically want public DNS.
- **Costs add up.** A big VPC with 15 interface endpoints × 3 AZs ≈ $330/mo idle.
- **VPC endpoint policies** can restrict which buckets / actions are allowed — useful for defense in depth.

## Related

- [VPC](./vpc.md)
- [S3](../02-storage/s3.md)
- [DynamoDB](../03-database/dynamodb.md)
