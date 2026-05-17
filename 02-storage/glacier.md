# S3 Glacier (Archive Storage Classes)

**TL;DR** — Cents-per-GB-year cold storage for backups, compliance archives, and rarely-accessed data. Slower retrievals, much cheaper.

## What it is

Glacier isn't a separate service anymore — it's three **S3 storage classes**:
- **Glacier Instant Retrieval** — instant access, like Standard-IA but cheaper.
- **Glacier Flexible Retrieval** — minutes to hours for retrieval; cheap.
- **Glacier Deep Archive** — 12-hour retrieval; cheapest tier on AWS.

## Pricing comparison

| Class | Storage $/GB-mo | Min storage duration | Retrieval |
|---|---|---|---|
| S3 Standard | $0.023 | — | instant |
| Glacier Instant | $0.004 | 90 days | instant |
| Glacier Flexible | $0.0036 | 90 days | minutes (Expedited) / 3-5 hrs (Standard) / 5-12 hrs (Bulk) |
| Glacier Deep Archive | $0.00099 | 180 days | 12-48 hrs |

Plus retrieval fees: $0.03/GB (Expedited), $0.01 (Standard), $0.0025 (Bulk).

## When to use which

- **Glacier Instant Retrieval** — Backups you rarely read but might need now.
- **Glacier Flexible Retrieval** — Long-term archives, occasional retrieval ok.
- **Glacier Deep Archive** — 7-10 year compliance archives, almost never retrieved.

## Real-world example

> A bank stores transaction logs for 7 years for regulators:
> - First 30 days in S3 Standard (active analytics).
> - Days 30-90 in Standard-IA.
> - Days 90-365 in Glacier Instant.
> - Years 1-7 in Glacier Deep Archive.
>
> Lifecycle rules do the moves automatically. Old data costs almost nothing.

## Usage

Glacier is just a **storage class** on S3 objects. Set via:
- Upload: `--storage-class GLACIER` (Flexible) / `DEEP_ARCHIVE` / `GLACIER_IR`.
- Lifecycle rule: auto-transition.

### Restore from Glacier (Flexible/Deep)

```bash
aws s3api restore-object \
  --bucket my-archive \
  --key 2019/transactions.parquet \
  --restore-request 'Days=7,GlacierJobParameters={Tier=Standard}'
```

After restore completes (hours later), object is available at S3 GET for `Days` days, then drops back to archive.

```bash
# Check restore status
aws s3api head-object --bucket my-archive --key 2019/transactions.parquet
# Returns x-amz-restore: ongoing-request="false", expiry-date="..."
```

## Gotchas

- **Minimum storage duration billing.** Delete a Glacier Deep object after 1 day → still charged 180 days.
- **Retrieval costs add up.** Reading a 1 TB Glacier archive = $25 (Standard tier).
- **No partial reads** in Flexible/Deep — must restore the whole object.
- **Glacier Instant has no restore step** — it acts like Standard-IA, just cheaper at scale.
- **Deep Archive minimum object size billed at 40 KB.** Don't archive millions of tiny files.

## Related

- [S3](./s3.md)
- [AWS Backup](#) — backup orchestrator (uses Glacier under the hood)
