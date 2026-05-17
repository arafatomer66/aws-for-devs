# IAM — Identity and Access Management

**TL;DR** — Who can do what to which AWS resources. Free. Global. **The single most important AWS concept to get right.** Get IAM wrong → either insecure or unproductive.

## What it is

IAM defines:
- **Identities** — who is acting (users, roles, federated principals).
- **Policies** — JSON documents describing allowed/denied actions.
- **Resources** — what's being acted on (specific S3 buckets, EC2 instances, etc.).
- **Conditions** — context restrictions (MFA, IP, region, time).

## Key concepts

- **Root user** — the account-owner email. Unlimited power. **Use only for setup**, then lock away with MFA.
- **IAM User** — a person/service with long-lived credentials. Increasingly discouraged; prefer roles.
- **IAM Group** — a bag of users; policies attached to the group apply to members.
- **IAM Role** — a set of permissions assumable by humans (via federation/SSO) or services (EC2, Lambda, ECS task).
- **Identity-based policy** — attached to user/role/group.
- **Resource-based policy** — attached to resources (S3 bucket policy, KMS key policy, SQS queue policy).
- **Permission boundary** — max permissions a role *can have* (delegation guardrail).
- **Service Control Policy (SCP)** — Organizations-level guardrail across an account.
- **Session policy** — passed at `AssumeRole` time to narrow further.
- **Access key** — `AKIA...` + secret. **Avoid for humans.** OK for service accounts when you must.
- **MFA** — required on root and recommended on all human users.
- **STS** — Security Token Service. Issues temporary creds for roles.

## How policies are evaluated

1. **Explicit Deny** anywhere → denied.
2. **SCP** denies → denied.
3. **Permission boundary** denies → denied.
4. **Identity or resource policy** allows → allowed.
5. **Otherwise** → denied (default deny).

## A policy looks like this

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ListMyBucket",
      "Effect": "Allow",
      "Action": ["s3:ListBucket"],
      "Resource": "arn:aws:s3:::my-bucket"
    },
    {
      "Sid": "ReadMyObjects",
      "Effect": "Allow",
      "Action": ["s3:GetObject"],
      "Resource": "arn:aws:s3:::my-bucket/*",
      "Condition": {
        "IpAddress": { "aws:SourceIp": "203.0.113.0/24" }
      }
    }
  ]
}
```

## The right way to authenticate

### Humans
- **Use IAM Identity Center (SSO)** with your identity provider (Okta, Google Workspace, Microsoft Entra).
- Users assume roles in target accounts via SSO.
- **No long-lived access keys for humans.**

### Workloads on AWS
- **EC2:** instance profile (role) → SDK auto-picks creds.
- **Lambda:** execution role.
- **ECS Fargate:** task role (for app code) + execution role (for ECS agent).
- **EKS:** IRSA or Pod Identity.

### Workloads outside AWS
- **OIDC / federation:** GitHub Actions → AWS via OIDC, no static keys.
- **IAM Roles Anywhere:** for on-prem servers using mTLS.

## Roles in code

When an EC2/Lambda/ECS picks up creds from its role, the SDK does it automatically — you don't write any code. Just call `S3Client()`; it figures it out.

```js
// On EC2/Lambda/ECS — no keys, just uses the attached role
const s3 = new S3Client({ region: "ap-south-1" });
```

## Creating things

### Create a role for Lambda

```bash
aws iam create-role --role-name LambdaReadS3 \
  --assume-role-policy-document '{
    "Version":"2012-10-17",
    "Statement":[{
      "Effect":"Allow",
      "Principal":{"Service":"lambda.amazonaws.com"},
      "Action":"sts:AssumeRole"
    }]
  }'

aws iam put-role-policy --role-name LambdaReadS3 --policy-name S3Read \
  --policy-document '{
    "Version":"2012-10-17",
    "Statement":[{"Effect":"Allow","Action":["s3:GetObject"],"Resource":"arn:aws:s3:::my-bucket/*"}]
  }'
```

### Attach a managed policy

```bash
aws iam attach-role-policy --role-name LambdaReadS3 \
  --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
```

## Least privilege patterns

- Start with denied; add only what's needed.
- Use **AWS managed policies** for common cases (e.g. `AWSLambdaBasicExecutionRole`).
- For app-specific: write a customer-managed policy listing exact actions/resources.
- Test with **IAM Access Analyzer** to find unused permissions.
- Use **Permission boundaries** when delegating role creation to teams.

## Common policy conditions

- `aws:MultiFactorAuthPresent` — require MFA.
- `aws:SourceIp` — restrict by IP.
- `aws:SourceVpc` / `aws:SourceVpce` — restrict to traffic from your VPC/endpoint.
- `aws:RequestedRegion` — limit to specific regions.
- `aws:PrincipalOrgID` — only your org.
- `aws:CalledVia` — restrict the chain of calls.

## Identity types — pick one

| You have… | Use… |
|---|---|
| Personal admin access | Identity Center → AdminAccess role |
| Team developers | Identity Center groups → role per team |
| Lambda function | Execution role |
| EC2 instance | Instance profile role |
| Fargate task | Task role + execution role |
| EKS pod | IRSA or Pod Identity |
| GitHub Actions / external CI | OIDC federated role |
| On-prem server | IAM Roles Anywhere |
| Mobile app users | Cognito Identity Pool (federated) |

## Real-world example

> ShareDeal's prod account IAM design:
>
> - **Root user** — MFA hardware key, in a safe. Never used after setup.
> - **Identity Center** for engineers — they SSO from Google Workspace, assume `Developer` or `ReadOnly` role in prod.
> - **Lambda functions** — each has its own role with only the perms it needs.
> - **ECS tasks** — task role with `dynamodb:Query` on specific tables only.
> - **GitHub Actions** — OIDC federation, assumes a `Deployer` role that can only update `selefe-*` resources.
> - **Permission boundary** on dev roles — they can't create IAM users or break out of dev guardrails.

## Gotchas

- **Don't put access keys in code, env vars committed to git, or unencrypted files.** Use roles.
- **Wildcards in resources** (`"Resource": "*"`) are a smell. Scope down to specific ARNs.
- **MFA required to delete** — set a policy condition.
- **Trust policy ≠ permission policy.** Trust says "who can assume this role?"; permission says "what can the role do?".
- **`PassRole`** is sneaky-powerful — needed to give a service a role. Restrict it carefully.
- **IAM is eventually consistent.** New users/roles/policies might take a few seconds to propagate.
- **AWS managed policies are sometimes too broad** — `AdministratorAccess` is admin god mode; `PowerUserAccess` is everything except IAM, which still includes a lot.
- **Service-linked roles** are auto-created by services; don't delete them blindly.

## Related

- [KMS](./kms.md)
- [Secrets Manager](./secrets-manager.md)
- [Identity Center](./identity-center.md)
- [Cognito](./cognito.md) — for *your end users*, not for your own team
- [Organizations](../11-cost-management/organizations.md)
