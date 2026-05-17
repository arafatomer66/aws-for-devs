# AWS Backup

**TL;DR** — Central backup orchestration. One service that backs up RDS / EBS / DynamoDB / EFS / S3 / FSx / Storage Gateway / Aurora / EC2 / Neptune / DocumentDB / Redshift / Timestream / VMware on-prem. Policy-driven, vault-encrypted.

## What it is

Instead of configuring backups per-service (RDS automated backups, EBS snapshots, DynamoDB PITR, etc.), AWS Backup applies one **backup plan** across many resources. Centralized monitoring, retention, cross-region/cross-account copies, and compliance reports.

## Key concepts

- **Backup vault** — KMS-encrypted store for recovery points.
- **Backup plan** — rules: which resources, how often, retention, copy destinations.
- **Selection** — resources matched by tags / ARNs.
- **Recovery point** — a single backup.
- **Vault lock** — WORM compliance (can't delete before retention even by root).
- **Cross-region / cross-account copy** — DR baked in.
- **Backup Audit Manager** — compliance reports (e.g., "all `Env=prod` resources backed up in last 24 h").

## Real-world example

> ShareDeal compliance + DR plan:
> - Tag everything in prod with `Backup=daily`.
> - Backup plan: daily at 02:00 UTC, retain 35 days, copy to `ap-southeast-1` vault for DR.
> - Vault lock: 35-day minimum retention — even compromised admin can't delete.
> - Audit report: nightly check that all `Env=prod` resources have a recovery point in last 25h.

## Usage

### Create a vault + plan

```bash
aws backup create-backup-vault --backup-vault-name prod-vault

aws backup create-backup-plan --backup-plan '{
  "BackupPlanName":"daily-35d",
  "Rules":[{
    "RuleName":"daily",
    "TargetBackupVaultName":"prod-vault",
    "ScheduleExpression":"cron(0 2 * * ? *)",
    "Lifecycle":{"DeleteAfterDays":35},
    "CopyActions":[{
      "DestinationBackupVaultArn":"arn:aws:backup:ap-southeast-1:..:backup-vault:dr-vault",
      "Lifecycle":{"DeleteAfterDays":35}
    }]
  }]
}'

# Select by tag
aws backup create-backup-selection --backup-plan-id <id> --backup-selection '{
  "SelectionName":"prod-resources",
  "IamRoleArn":"arn:aws:iam::..:role/AWSBackupDefaultServiceRole",
  "ListOfTags":[{"ConditionType":"STRINGEQUALS","ConditionKey":"Backup","ConditionValue":"daily"}]
}'
```

### Restore

```bash
aws backup start-restore-job \
  --recovery-point-arn arn:aws:backup:..:recovery-point:... \
  --metadata <service-specific JSON> \
  --iam-role-arn arn:aws:iam::..:role/AWSBackupRestoreRole
```

## Pricing

- **No charge for Backup itself.**
- You pay for the underlying storage:
  - EBS snapshots: $0.05/GB-mo.
  - RDS snapshots: similar.
  - S3 backups: backup-specific storage class fee.
  - EFS backups: $0.05/GB-mo cold, $0.30 warm.
- Cross-region copy: data transfer charges.
- Restore: per-GB read.

## Gotchas

- **Some services need an opt-in** in the AWS Backup console (`Settings → Service opt-in`).
- **PITR is separate** — DynamoDB PITR isn't replaced by Backup; both exist.
- **Vault lock is permanent in Compliance mode** — even AWS support can't remove. Be sure.
- **Cross-account backup** needs Organizations + delegated admin.
- **Restore tests** — actually try restoring at least quarterly. Untested backups aren't backups.

## Related

- [EBS](./ebs.md)
- [EFS](./efs.md)
- [S3](./s3.md)
- [RDS](../03-database/rds.md)
- [DynamoDB](../03-database/dynamodb.md)
