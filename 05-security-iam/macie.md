# Macie

**TL;DR** — ML-based PII / sensitive data discovery in S3. Finds credit cards, SSNs, secrets, addresses in your buckets. Reports findings to Security Hub.

## What it is

Scans S3 objects (JSON, CSV, PDF, plain text, etc.) for sensitive data patterns: PII, financial data, credentials. Maintains an inventory of S3 buckets, their public exposure, encryption status, and content classification.

## Key concepts

- **Sensitive data discovery job** — one-time or recurring scan.
- **Managed data identifiers** — ~150 prebuilt patterns (SSN, IBAN, AWS access key, etc.).
- **Custom data identifiers** — your own regex with proximity rules.
- **Findings** — `SensitiveData:S3Object/Personal`, `Policy:IAMUser/S3BlockPublicAccessDisabled`, etc.
- **Allow lists** — file paths or regex to exclude from scanning.

## Real-world example

> Compliance audit:
> - Enable Macie account-wide.
> - Run a discovery job across all S3 buckets.
> - Macie finds: 12k objects with credit card patterns in a forgotten log bucket.
> - Investigation → fix root cause + redact + restrict bucket access.

## Usage

```bash
aws macie2 enable-macie

aws macie2 create-classification-job --job-type ONE_TIME \
  --name pii-audit \
  --s3-job-definition 'bucketDefinitions=[{accountId="123456789012",buckets=["my-bucket"]}]'

aws macie2 get-findings --finding-ids ...
```

## Pricing

- **Bucket inventory:** small fee per bucket per month.
- **Sensitive data discovery:** ~$1 per GB scanned (first 1 GB free).

Scans can be expensive on huge buckets — use sampling or scope to high-risk prefixes.

## Gotchas

- **Per-region service.** Findings are per region.
- **Initial scan is the costly one** — incremental scans cheaper.
- **False positives** — tune custom identifiers.
- **Doesn't auto-remediate** — pair with EventBridge → Lambda for actions.

## Related

- [Security Hub](./security-hub.md)
- [GuardDuty](./guardduty.md)
- [Inspector](./inspector.md)
- [S3](../02-storage/s3.md)
