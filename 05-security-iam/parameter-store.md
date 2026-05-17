# SSM Parameter Store

**TL;DR** — Free key/value config store. Hierarchical names like `/app/prod/db-url`. Supports SecureString (KMS-encrypted). Cheap-or-free alternative to Secrets Manager for non-rotating secrets.

## What it is

Part of AWS Systems Manager. Stores configuration parameters (strings, lists, SecureStrings) with hierarchical naming.

## Tiers

- **Standard** — 4 KB per parameter, free, 10,000 parameters per account.
- **Advanced** — 8 KB, $0.05/param/mo, supports policies (expiration, notification), higher throughput.

## Real-world example

> Per-service config:
> - `/sd/prod/api/log-level` = `"info"` (String)
> - `/sd/prod/api/feature-flags` = JSON (String)
> - `/sd/prod/api/db-password` = ciphertext (SecureString w/ KMS)
> - `/sd/staging/api/...` mirror for staging

App reads `/sd/${ENV}/api/*` at startup with one `GetParametersByPath` call.

## Usage

### Create

```bash
aws ssm put-parameter --name /sd/prod/api/log-level --value info --type String
aws ssm put-parameter --name /sd/prod/api/db-password --value 's3cret!' \
  --type SecureString --key-id alias/aws/ssm
```

### Read

```bash
aws ssm get-parameter --name /sd/prod/api/log-level --query Parameter.Value --output text
aws ssm get-parameter --name /sd/prod/api/db-password --with-decryption --query Parameter.Value --output text

# Get a whole path at once
aws ssm get-parameters-by-path --path /sd/prod/api/ --with-decryption --recursive
```

### Read from app (Node)

```js
import { SSMClient, GetParametersByPathCommand } from "@aws-sdk/client-ssm";
const ssm = new SSMClient({ region: "ap-south-1" });
const { Parameters } = await ssm.send(new GetParametersByPathCommand({
  Path: "/sd/prod/api/", Recursive: true, WithDecryption: true,
}));
const config = Object.fromEntries(Parameters.map(p => [p.Name.split("/").pop(), p.Value]));
```

### Inject into ECS (no SDK needed)

```json
"secrets": [
  { "name": "DB_PASS", "valueFrom": "arn:aws:ssm:ap-south-1:..:parameter/sd/prod/api/db-password" }
]
```

## Pricing

- **Standard:** free for storage + 10k API calls/mo.
- **Advanced:** $0.05/param/mo + $0.05 per 10k calls.

For most apps: completely free.

## When to use Parameter Store vs Secrets Manager

| Need | Pick |
|---|---|
| Rotating DB passwords | Secrets Manager |
| Static API keys for third parties | Parameter Store |
| App feature flags / config | Parameter Store |
| High API throughput | Parameter Store Advanced |
| Cross-region replication | Secrets Manager |

## Gotchas

- **Standard tier has lower API throughput** — burst limits exist.
- **No native rotation.** Build your own with Lambda + EventBridge if needed.
- **SecureString reads must specify `--with-decryption`** or you get ciphertext.
- **Hierarchical names** rule — design `/team/env/service/param`.
- **Param values are cleartext to anyone with read IAM** — control who has the IAM action.

## Related

- [Secrets Manager](./secrets-manager.md)
- [Systems Manager](../07-devops-iac/systems-manager.md)
- [KMS](./kms.md)
