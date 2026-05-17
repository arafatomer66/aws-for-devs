# AWS Config

**TL;DR** — Continuously records resource configuration over time, evaluates against compliance rules. "Was this bucket public on March 1st?" — Config knows.

## What it is

A configuration recorder + rules engine. Tracks every resource's settings (Configuration Items) and lets you evaluate rules ("S3 buckets must be encrypted", "Security groups must not allow 0.0.0.0/0 to port 22").

## Key concepts

- **Configuration recorder** — opts your account into Config.
- **Configuration item (CI)** — snapshot of a resource at a point in time.
- **Resource history** — versioned config over time.
- **Rule** — managed or custom (Lambda); evaluates compliance.
- **Conformance pack** — bundle of rules (CIS, PCI, HIPAA, NIST templates).
- **Aggregator** — combine data from multiple accounts/regions.
- **Remediation** — auto-fix via Systems Manager Automation.

## Real-world example

> Compliance team wants:
> - All S3 buckets encrypted with KMS.
> - No public-facing security groups on port 22.
> - All EBS volumes encrypted.
>
> Enable Config managed rules: `s3-bucket-server-side-encryption-enabled`, `restricted-ssh`, `encrypted-volumes`. Non-compliant findings show in Security Hub. Remediation auto-runs to encrypt new volumes.

## Usage

```bash
# Enable Config (recorder + delivery channel)
aws configservice put-configuration-recorder \
  --configuration-recorder name=default,roleARN=arn:aws:iam::..:role/AWSConfigRole \
  --recording-group allSupported=true,includeGlobalResourceTypes=true

aws configservice put-delivery-channel \
  --delivery-channel name=default,s3BucketName=my-config-bucket

aws configservice start-configuration-recorder --configuration-recorder-name default

# Add a managed rule
aws configservice put-config-rule --config-rule '{
  "ConfigRuleName": "s3-encryption",
  "Source": {"Owner":"AWS","SourceIdentifier":"S3_BUCKET_SERVER_SIDE_ENCRYPTION_ENABLED"}
}'
```

## Pricing

- **CI:** $0.003 per CI recorded.
- **Rule evaluation:** $0.001 per evaluation.
- **Aggregator:** small per-resource fee.

Big accounts can rack up surprising Config bills. Be selective with `allSupported=true`.

## Gotchas

- **Per-region.** Enable in every region you use.
- **First CI batch is huge** at enablement.
- **Doesn't fix anything by default** — pair with Remediation.
- **Custom rules** are Lambdas — pay Lambda invocation cost.
- **Records all** by default; filter via recording group.

## Related

- [Security Hub](../05-security-iam/security-hub.md)
- [CloudTrail](./cloudtrail.md)
- [Systems Manager Automation](../07-devops-iac/systems-manager.md)
