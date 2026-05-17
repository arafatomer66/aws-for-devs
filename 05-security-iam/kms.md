# KMS — Key Management Service

**TL;DR** — Managed cryptographic keys. AWS services use KMS to encrypt at rest, your app can use it for envelope encryption. $1/key/month for customer keys; AWS-managed keys are free.

## What it is

A managed service for creating + using encryption keys without ever exposing the raw key material. Operates inside FIPS-validated HSMs. You call `Encrypt` / `Decrypt` / `GenerateDataKey` via API; AWS handles the rest.

## Key types

- **AWS-owned keys** — invisible, used by AWS for default SSE-S3 etc. Free.
- **AWS-managed keys** (`aws/<service>`) — visible in your account, AWS controls. Free.
- **Customer-managed keys (CMK)** — you create/control/audit/rotate. $1/key/mo + $0.03 per 10k requests.
- **Imported key material** — you BYOK; AWS holds the wrapping key.
- **CloudHSM keys** — keys in your own dedicated HSM cluster (separate service).

Key specs:
- **Symmetric** (AES-256-GCM) — most common.
- **Asymmetric** — RSA, ECC. For signing or hybrid encrypt.
- **HMAC** — for MAC generation.

## Envelope encryption (the canonical pattern)

1. App asks KMS for a **data key** — KMS returns plaintext key + ciphertext key.
2. App encrypts data with plaintext data key (fast, local).
3. App discards plaintext data key; stores ciphertext + ciphertext data key together.
4. To decrypt: KMS decrypts the ciphertext data key → app uses it to decrypt the data.

This way you never call KMS for every byte — only per data-key generation.

## Real-world example

> ShareDeal encrypts PII fields (phone, NID) in the database:
> - One CMK per environment: `alias/sd-prod-pii`.
> - App calls `GenerateDataKey` per record; encrypts the field with AES-GCM, stores ciphertext + the encrypted data key.
> - All operations logged in CloudTrail with which IAM role accessed which key.

## Usage

### Create a CMK

```bash
aws kms create-key --description "Prod PII" --key-usage ENCRYPT_DECRYPT --key-spec SYMMETRIC_DEFAULT
aws kms create-alias --alias-name alias/sd-prod-pii --target-key-id <key-id>
```

### Encrypt / Decrypt (small payloads only, < 4 KB)

```bash
aws kms encrypt --key-id alias/sd-prod-pii --plaintext "secret stuff" \
  --query CiphertextBlob --output text | base64 -d > cipher.bin
aws kms decrypt --ciphertext-blob fileb://cipher.bin --query Plaintext --output text | base64 -d
```

### Envelope encryption (Node SDK)

```js
import { KMSClient, GenerateDataKeyCommand, DecryptCommand } from "@aws-sdk/client-kms";
import { createCipheriv, createDecipheriv, randomBytes } from "crypto";

const kms = new KMSClient({ region: "ap-south-1" });

// Encrypt
async function encrypt(plaintext) {
  const { Plaintext, CiphertextBlob } = await kms.send(new GenerateDataKeyCommand({
    KeyId: "alias/sd-prod-pii",
    KeySpec: "AES_256",
  }));
  const iv = randomBytes(12);
  const cipher = createCipheriv("aes-256-gcm", Plaintext, iv);
  const ciphertext = Buffer.concat([cipher.update(plaintext, "utf8"), cipher.final()]);
  const tag = cipher.getAuthTag();
  return { encryptedKey: CiphertextBlob, iv, tag, ciphertext };
}

// Decrypt
async function decrypt({ encryptedKey, iv, tag, ciphertext }) {
  const { Plaintext } = await kms.send(new DecryptCommand({ CiphertextBlob: encryptedKey }));
  const decipher = createDecipheriv("aes-256-gcm", Plaintext, iv);
  decipher.setAuthTag(tag);
  return Buffer.concat([decipher.update(ciphertext), decipher.final()]).toString("utf8");
}
```

## Key policy (resource-based)

Defines who can use the key. Must explicitly grant your account or principals:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Enable IAM",
      "Effect": "Allow",
      "Principal": { "AWS": "arn:aws:iam::123456789012:root" },
      "Action": "kms:*",
      "Resource": "*"
    },
    {
      "Sid": "Allow Lambda role",
      "Effect": "Allow",
      "Principal": { "AWS": "arn:aws:iam::123456789012:role/lambda-app" },
      "Action": ["kms:Decrypt", "kms:GenerateDataKey"],
      "Resource": "*"
    }
  ]
}
```

## Pricing

- **CMK:** $1/mo per key.
- **Requests:** $0.03 per 10k.
- **AWS managed keys:** free (but you can't customize policy or audit them as deeply).

## Rotation

- **CMK auto-rotation** — toggle on; AWS rotates yearly automatically. Old ciphertext still decrypts with old material.
- **Manual rotation** — create new key; re-encrypt existing data; update aliases.

## Where you'll see KMS

- **S3** — SSE-KMS encryption.
- **EBS** — encrypted volumes.
- **RDS / Aurora** — at-rest encryption.
- **DynamoDB** — managed or CMK encryption.
- **SQS / SNS** — message encryption.
- **Secrets Manager / Parameter Store** — wraps secrets with KMS.
- **CloudWatch Logs** — log group encryption.

## Gotchas

- **Can't delete a key immediately.** 7-30 day waiting period (`ScheduleKeyDeletion`).
- **Each request is rate-limited** — default 5,500-30,000 per second per key region.
- **Cross-region**: keys are regional; you can replicate a multi-region key now.
- **Don't store encrypted data without the key reference** — losing the key = losing the data.
- **Key policy + IAM policy must both allow access** (unless the key policy grants the account at the root).
- **Some services need the key in a specific account** (cross-account access requires grants).

## Related

- [Secrets Manager](./secrets-manager.md)
- [S3](../02-storage/s3.md) — SSE-KMS
- [IAM](./iam.md)
