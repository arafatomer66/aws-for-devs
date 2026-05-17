# CloudTrail

**TL;DR** — Logs every API call in your AWS account (control plane + optional data plane). Your security/audit/forensics record. **Enable in every account, every region.**

## What it is

A managed audit log. Records `who called what API, when, from where, with what parameters`. Stored in S3 (and optionally CloudWatch Logs or EventBridge events).

## Key concepts

- **Event** — one API call.
- **Trail** — config that delivers events to S3.
- **Multi-region trail** — captures all regions in one.
- **Organization trail** — covers all accounts in an Organization.
- **Management events** — control-plane (Create/Modify/Delete resources). Free first copy.
- **Data events** — data-plane (S3 GetObject, Lambda Invoke, DynamoDB GetItem). Opt-in. Costs $$.
- **Insights events** — anomaly detection on the trail itself.
- **CloudTrail Lake** — SQL-queryable archive of events.

## What's in an event

- `eventName`, `eventSource`, `eventTime`
- `userIdentity` (IAM user/role)
- `sourceIPAddress`, `userAgent`
- `requestParameters`, `responseElements`
- `awsRegion`, `errorCode` (if denied)

## Real-world example

> Audit: "Who deleted the production S3 bucket at 02:14 UTC?"
> - Query CloudTrail (Lake or Athena over S3):
> ```sql
> SELECT eventTime, userIdentity.arn, sourceIPAddress
> FROM cloudtrail
> WHERE eventName = 'DeleteBucket'
>   AND requestParameters.bucketName = 'selefe-uploads-prod'
> ```
> - Find the offending role, investigate.

## Usage

### Create a trail (multi-region, all events to S3)

```bash
aws cloudtrail create-trail --name org-trail \
  --s3-bucket-name my-audit-logs \
  --is-multi-region-trail \
  --enable-log-file-validation

aws cloudtrail start-logging --name org-trail
```

### Query with Athena

CloudTrail console can auto-create an Athena table over your S3 logs. Then:
```sql
SELECT eventTime, eventName, userIdentity.arn
FROM cloudtrail_logs
WHERE eventTime > '2026-05-17T00:00:00Z'
  AND errorCode IS NOT NULL
ORDER BY eventTime DESC LIMIT 100;
```

### CloudTrail Lake

```bash
aws cloudtrail create-event-data-store --name lake-store --retention-period 365
# Query via the CloudTrail Lake UI or API with SQL
```

## Pricing

- **Management events:** first copy free, additional trails $2.00/100k events.
- **Data events:** $0.10/100k events (can balloon — careful with S3/Lambda data events).
- **Insights:** $0.35 per 100k events analyzed.
- **CloudTrail Lake:** ingestion $2.50/GB + storage $0.025/GB-mo.

## Gotchas

- **Always-on management events.** Even without a trail, the last 90 days are queryable in **Event History** (console).
- **Data events not on by default** — turn on for S3 buckets with sensitive content.
- **Cross-region trails** are not on by default for new accounts; verify.
- **S3 log delivery is eventually consistent** — events may appear minutes later.
- **Log file validation** — turn on; lets you detect tampering.
- **CloudTrail itself doesn't alert** — feed events to EventBridge for real-time alerting.

## Real-time alerting via EventBridge

```json
{
  "source": ["aws.cloudtrail"],
  "detail-type": ["AWS API Call via CloudTrail"],
  "detail": { "eventName": ["DeleteBucket", "TerminateInstances"] }
}
```
Target: SNS → on-call.

## Related

- [CloudWatch](./cloudwatch.md)
- [Athena](../10-analytics/athena.md) — query CloudTrail logs
- [Security Hub](../05-security-iam/security-hub.md)
- [GuardDuty](../05-security-iam/guardduty.md)
