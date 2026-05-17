# AWS Accounts, Billing & Free Tier

**TL;DR** — An AWS account is a security + billing boundary. **Don't use the root user.** Use IAM users (or better, IAM Identity Center / SSO) for day-to-day work.

## What is an AWS account?

A single AWS account has:
- **One root user** (the email you signed up with).
- A **billing address** and credit card.
- An **account ID** (12 digits, e.g. `123456789012`).
- All resources you create — VPCs, S3 buckets, Lambda functions — live inside *this* account by default.

Multi-account is normal for real orgs (one for prod, one for staging, one for sandbox, etc.), managed via **AWS Organizations**.

## Day-1 setup checklist

1. **Sign up** at aws.amazon.com → confirm email → enter card.
2. **Enable MFA on the root user.** Use a hardware key or authenticator app. Then put the root credentials away.
3. **Create an IAM user** (or Identity Center user) with admin permissions. Use this for daily work.
4. **Set up Billing alerts** — `Budgets` → notify at $10 / $50 / $100.
5. **Enable IAM Access Analyzer.**
6. **Enable CloudTrail** in all regions.
7. **Set up `aws configure`** locally with the IAM user's access keys.

```bash
# After creating the IAM user
aws configure
# AWS Access Key ID: AKIA...
# AWS Secret Access Key: ...
# Default region name: ap-south-1
# Default output format: json
```

## The Free Tier

Three flavors:

| Type | Examples | How long |
|---|---|---|
| **12-months free** | 750 hrs/mo of t2.micro / t3.micro EC2, 5 GB S3, 750 hrs of RDS | 12 months from signup |
| **Always free** | 1M Lambda invocations/mo, 25 GB DynamoDB, 62k SNS publishes/mo, 1M SQS requests/mo | Forever |
| **Trials** | SageMaker (2 mo), Lightsail (3 mo) | One-time |

**Reality check:** Free tier covers a hobby project but **does not cover production**. NAT Gateway alone is ~$32/month and never free. CloudWatch logs add up fast. Always set a Budget alert.

## Where bills come from

Charges roll up into categories:
- **Compute** (EC2, Lambda, Fargate).
- **Storage** (S3 storage + requests, EBS volumes).
- **Data transfer** — this is the silent killer:
  - Inbound: **free**.
  - Within same AZ: **free**.
  - Between AZs in same Region: **$0.01/GB each way**.
  - Out to internet: **~$0.09/GB** (decreasing with volume).
  - Between Regions: **~$0.02/GB**.
- **NAT Gateway**: ~$0.045/hr + $0.045/GB processed.
- **CloudWatch**: logs ingestion is $0.50/GB, custom metrics $0.30 each.

## Reading the bill

```
AWS Console → Billing & Cost Management → Cost Explorer
```

- **Group by Service** — find the top offenders.
- **Group by Tag** — if you tagged your resources (you should!), see cost by `Project=ShareDeal` or `Env=Prod`.
- **Group by Usage Type** — distinguishes "EC2 hours" from "EC2 data transfer."

## Tagging strategy (do this early)

Tags are key-value pairs on resources. **Without tags, you can't tell who's spending what.**

Suggested minimum:
- `Project` — `sharedeal-app`
- `Env` — `prod` | `staging` | `dev`
- `Owner` — `team-payments`
- `CostCenter` — for billing rollups

Activate them as **Cost Allocation Tags** in Billing settings so they appear in Cost Explorer.

## Multi-account (AWS Organizations)

For anything beyond a personal project, use multiple accounts:
- `mgmt` — only Organizations + billing lives here, no workloads.
- `prod` — production workloads.
- `staging` — pre-prod.
- `dev` / `sandbox` — playgrounds.
- `security` — central CloudTrail, GuardDuty findings, log archive.

This is what **AWS Control Tower** automates (landing zone).

## Gotchas

- **Root user has unlimited power and can't be IAM-restricted.** Lock it down with MFA and never use it.
- **Free tier doesn't include data transfer to internet.** A million `getObject` calls returning images can be free for storage but expensive for egress.
- **Budgets only notify, they don't stop spend.** Use Service Quotas or shut down resources if you need a hard cap.
- **Card declines pause your account.** Keep billing email valid.
- **Inactive resources still cost money** — unattached EBS volumes, idle NAT Gateways, orphaned ELBs, old EIPs (Elastic IPs not attached cost $0.005/hr each).

## Related

- [IAM](../05-security-iam/iam.md)
- [Pricing Models](./05-pricing-models.md)
- [Cost Explorer](../11-cost-management/cost-explorer.md)
- [Organizations](../11-cost-management/organizations.md)
