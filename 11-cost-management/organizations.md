# AWS Organizations

**TL;DR** — Manage many AWS accounts as one org. Consolidated billing, Service Control Policies (SCPs), shared services. The foundation for any serious AWS deployment.

## What it is

A management layer over multiple AWS accounts. One **management account** (formerly "master") owns the org; **member accounts** are subordinate. Lets you:
- Bill all accounts to one invoice (volume discounts roll up).
- Apply guardrails (SCPs) to all or some accounts.
- Centrally enable services (Config, Security Hub, GuardDuty, Identity Center).
- Share resources via RAM (Resource Access Manager).

## Why use multiple accounts

- **Blast radius** — IAM mistakes in dev can't touch prod.
- **Compliance** — separate accounts for regulated workloads.
- **Billing clarity** — per-account costs visible without complex tag schemes.
- **Quotas** — service quotas are per-account; spread workloads to avoid limits.

## Typical multi-account layout

```
Organization
├── Management (only billing + Organizations)
├── Security (CloudTrail logs, central GuardDuty)
├── Log Archive (long-term audit logs)
├── Shared Services (DNS, CI/CD, central artifact repo)
├── Network (TGW, central VPCs)
├── Production
│   ├── prod-app1
│   └── prod-app2
├── Staging
│   ├── staging-app1
│   └── staging-app2
└── Sandbox
    ├── alice-sandbox
    └── bob-sandbox
```

## Key concepts

- **Management account** — pays the bills, can't be member-of itself.
- **Member account** — subordinate; can be moved between OUs.
- **OU (Organizational Unit)** — a group of accounts for policy targeting.
- **SCP (Service Control Policy)** — guardrails (DENY-only effectively). E.g., "no account can disable CloudTrail."
- **RAM (Resource Access Manager)** — share resources (TGW, subnets, prefix lists) across accounts.
- **Trusted services** — services that can act org-wide (CloudTrail, Config, Identity Center, Security Hub, GuardDuty).

## SCP example

Deny disabling CloudTrail:
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Deny",
    "Action": ["cloudtrail:DeleteTrail","cloudtrail:StopLogging"],
    "Resource": "*"
  }]
}
```

Attach to OUs you want to restrict.

## Real-world example

> A startup grows from 1 to 4 accounts:
> - `mgmt` (billing + Organizations).
> - `prod` (live app).
> - `staging`.
> - `dev` (engineer sandboxes).
> - Identity Center connected to Google Workspace.
> - All four accounts consolidated into one bill with cost-allocation tags.
> - SCPs prevent disabling CloudTrail and using regions outside `ap-south-1` + `us-east-1`.

## Usage

Mostly console-driven for setup. Then:

```bash
aws organizations create-account --account-name dev-account --email dev@mycompany.com
aws organizations move-account --account-id 123456789012 --source-parent-id ou-root --destination-parent-id ou-dev
aws organizations attach-policy --policy-id p-xxxx --target-id ou-dev
```

## Pricing

- **Organizations: free.**
- You pay for whatever resources accounts create. **Volume discounts** kick in automatically (S3 storage tiers, data transfer tiers, etc.).

## Control Tower vs DIY Organizations

- **Control Tower** = AWS's opinionated "landing zone" — automates a baseline (logging, guardrails, account factory).
- **DIY** = build your own with raw Organizations + CDK/Terraform.

For >5 accounts, Control Tower (or terraform `aws-landing-zone` patterns) is worth it.

## Gotchas

- **Management account is special.** Don't run workloads here; only Organizations + Billing.
- **SCPs deny only.** They don't grant permissions — IAM still does.
- **Closing an account** — 90-day grace, then permanent. Don't accidentally close.
- **Identity Center needs to be set up in the management account** in a specific region.
- **Email per account must be unique** — use `+aliases` if your domain allows.

## Related

- [IAM](../05-security-iam/iam.md)
- [Identity Center](../05-security-iam/identity-center.md)
- [Control Tower](./control-tower.md)
- [Cost Explorer](./cost-explorer.md)
