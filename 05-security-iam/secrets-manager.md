# Secrets Manager

**TL;DR** — Store secrets (DB passwords, API keys), retrieve via API, rotate automatically. ~$0.40/secret/month + API costs.

## What it is

A managed vault for secrets. Versioned, KMS-encrypted, IAM-controlled. Can auto-rotate by triggering a Lambda function.

## Why it exists

You used to bake secrets into env vars, `.env` files, or worse, source code. Secrets Manager centralizes them, audits access, and rotates them on a schedule.

## Key concepts

- **Secret** — named blob (JSON or raw string).
- **Version** — each `PutSecretValue` creates a new version. Stages (`AWSCURRENT`, `AWSPREVIOUS`, `AWSPENDING`) track active vs old.
- **Rotation** — Lambda-driven; for RDS/Aurora/Redshift/DocDB, AWS provides ready-made rotation lambdas.
- **Resource policy** — restrict who can read.
- **Replication** — replicate to other regions.

## Real-world example

> A Postgres DB password:
> - Secret `sd-prod-db-password` holds `{username, password, host, port, dbname}`.
> - Lambda rotates the password every 30 days using the AWS-provided rotation template (single-user pattern).
> - The app reads the secret on startup + on cached-cert expiry.

## Usage

### Create

```bash
aws secretsmanager create-secret --name sd-prod-db-password \
  --secret-string '{"username":"sdadmin","password":"s3cret!","host":"db.xxx.rds.amazonaws.com","port":5432,"dbname":"prod"}'
```

### Retrieve (CLI)

```bash
aws secretsmanager get-secret-value --secret-id sd-prod-db-password \
  --query SecretString --output text
```

### Retrieve (Node)

```js
import { SecretsManagerClient, GetSecretValueCommand } from "@aws-sdk/client-secrets-manager";
const sm = new SecretsManagerClient({ region: "ap-south-1" });
const { SecretString } = await sm.send(new GetSecretValueCommand({ SecretId: "sd-prod-db-password" }));
const { username, password, host } = JSON.parse(SecretString);
```

### Inject into ECS/Lambda env (no SDK calls needed)

ECS task definition:
```json
"secrets": [
  { "name": "DB_PASS", "valueFrom": "arn:aws:secretsmanager:ap-south-1:..:secret:sd-prod-db-password:password::" }
]
```

Lambda: same syntax in console / SAM / CDK. Or use the Lambda Extension for caching.

### Enable rotation (RDS pattern)

```bash
aws secretsmanager rotate-secret \
  --secret-id sd-prod-db-password \
  --rotation-lambda-arn arn:aws:lambda:ap-south-1:123:function:SecretsManagerRDSPostgreSQLRotationSingleUser \
  --rotation-rules AutomaticallyAfterDays=30
```

## Pricing

- **$0.40 per secret per month.**
- **$0.05 per 10,000 API calls.**

For 50 secrets at $0.40 = $20/mo. Cache reads to avoid request blowup.

## Secrets Manager vs Parameter Store

| | Secrets Manager | SSM Parameter Store (SecureString) |
|---|---|---|
| Cost | $0.40/secret/mo | Free for standard, $0.05/param/mo advanced |
| Built-in rotation | Yes | No |
| Replication | Yes | No |
| Versioning | Yes | Limited |
| Hierarchical names | Limited | Yes (`/app/prod/db`) |
| API throughput | Higher | Lower for standard |

Use Parameter Store for **config** (low-stakes, infrequent). Use Secrets Manager for **secrets that rotate** (DB creds, API keys).

## Gotchas

- **Cache reads** to avoid the $0.05/10k charge piling up. The Lambda Extension does this for you.
- **Rotation lambdas fail silently** if VPC config is wrong (can't reach DB). Test in non-prod.
- **Cross-region replication** — replicas count as additional secrets ($0.40 each).
- **Don't fetch on every request.** Read on startup; refresh on auth failure.
- **Soft-deleted secrets** wait 7-30 days before permanent delete.

## Related

- [Parameter Store](./parameter-store.md)
- [KMS](./kms.md)
- [IAM](./iam.md)
