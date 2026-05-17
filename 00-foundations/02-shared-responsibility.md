# Shared Responsibility Model

**TL;DR** — AWS secures the cloud (hardware, hypervisor, facilities). **You** secure what's *in* the cloud (your data, IAM policies, OS patches if you manage them, app code).

## The split

```
┌─────────────────────────────────────────────────────────┐
│  YOUR responsibility — security IN the cloud            │
│                                                         │
│  • Customer data                                        │
│  • IAM users, roles, policies                           │
│  • Network/firewall config (security groups, NACLs)     │
│  • App code & dependencies                              │
│  • OS patches (for EC2)                                 │
│  • Encryption choices (KMS keys, TLS)                   │
└─────────────────────────────────────────────────────────┘
┌─────────────────────────────────────────────────────────┐
│  AWS responsibility — security OF the cloud             │
│                                                         │
│  • Physical data centers                                │
│  • Hardware & networking                                │
│  • Hypervisor                                           │
│  • Managed service runtime (for Lambda, DynamoDB, etc.) │
└─────────────────────────────────────────────────────────┘
```

## The line moves depending on the service

The more "managed" the service, the less you handle.

| Service type | Your job ends at… |
|---|---|
| **EC2** (IaaS) | OS + everything above. You patch the kernel. |
| **RDS** (managed DB) | DB config, users, schemas. AWS patches the engine. |
| **Lambda / DynamoDB / S3** (serverless / fully managed) | Just your code, IAM, and data. AWS does the rest. |

## What this means in practice

**You are responsible for:**
1. **IAM** — least-privilege roles, no root key in code, MFA on root.
2. **S3 bucket policies** — most public-data leaks are misconfigured S3 buckets.
3. **Security groups** — your "firewall." Don't open 0.0.0.0/0 to port 22.
4. **Encryption** — enable at-rest encryption on S3/EBS/RDS (it's free, just a checkbox).
5. **App-level auth** — Cognito, JWT, OAuth — AWS doesn't authenticate your end users for you.
6. **Audit logging** — turn on CloudTrail in every account, every region.
7. **Vulnerability scanning** — Inspector for EC2/ECR images, dependency scanning in your CI.

**AWS handles:**
- Power, cooling, physical access to data centers.
- The hypervisor (Nitro).
- Service infrastructure (Lambda's container runtime, DynamoDB's storage layer).
- Patching of managed services.

## Real-world example

> A company stores customer PII in S3. They:
> - Use AWS-managed KMS keys → AWS secures the key infrastructure ✅
> - Set the bucket to public-read by accident → **their fault**, data leaks ❌
>
> AWS will not refund you for a leak caused by your own bucket policy.

## The "Shared Controls" middle ground

Some areas are shared:
- **Patch management** — AWS patches Lambda's runtime; you patch your EC2 AMIs.
- **Configuration management** — AWS configures the hardware; you configure security groups.
- **Awareness & training** — AWS trains its staff; you train yours.

## Gotchas

- **Default isn't always secure.** New S3 buckets are private now (since 2023), but older buckets, EBS snapshots, AMIs, RDS snapshots, etc., can still be made public by mistake.
- **Compliance certifications cover AWS's part only.** SOC 2 / ISO 27001 from AWS doesn't make your app compliant — you still need to do your half.
- **Region-specific compliance:** GDPR, HIPAA, PCI-DSS — make sure both you and AWS in that Region meet the standard.

## Related

- [IAM](../05-security-iam/iam.md)
- [Well-Architected Framework](./03-well-architected.md) — Security pillar
- [CloudTrail](../08-monitoring-observability/cloudtrail.md)
