# LocalStack

**TL;DR** — Run AWS locally (Docker container). Mocks S3, DynamoDB, Lambda, SQS, SNS, KMS, etc. Great for offline dev + CI. Third-party but ubiquitous.

## What it is

A community + commercial project that emulates AWS APIs in a Docker container. Point your AWS SDK at `http://localhost:4566`; CRUD a fake S3 bucket without an AWS account.

Tiers:
- **Community (free)** — S3, DynamoDB, SQS, SNS, Lambda, Kinesis, IAM (partial), KMS, ApiGW, SES, CloudWatch, more.
- **Pro/Teams/Enterprise (paid)** — more services, advanced features (CloudFormation drift, replay, Lambda multi-arch, RDS via Postgres backend, etc.).

## Why use it

- **Fast inner loop** — no network, no AWS bills, no shared sandbox conflicts.
- **CI tests** — run integration tests against fake AWS in GitHub Actions.
- **Offline dev** — on a plane.

## Usage

### Run

```bash
docker run --rm -d -p 4566:4566 -p 4510-4559:4510-4559 \
  -e SERVICES=s3,dynamodb,sqs,sns \
  localstack/localstack
```

Or use Docker Compose, or `localstack start` from their CLI.

### Point SDK at it

```bash
export AWS_ACCESS_KEY_ID=test
export AWS_SECRET_ACCESS_KEY=test
export AWS_DEFAULT_REGION=us-east-1
export AWS_ENDPOINT_URL=http://localhost:4566
aws s3 mb s3://my-bucket
aws s3 cp file.txt s3://my-bucket/
aws s3 ls s3://my-bucket/
```

### In code (Node SDK v3)

```js
import { S3Client } from "@aws-sdk/client-s3";
const s3 = new S3Client({
  endpoint: process.env.AWS_ENDPOINT_URL,   // points to LocalStack if set
  region: "us-east-1",
  forcePathStyle: true,
});
```

## Real-world example

> ShareDeal integration tests use LocalStack:
> - `docker compose up` starts LocalStack + Postgres locally.
> - Test setup creates a DynamoDB table + SQS queue in LocalStack.
> - Tests run against the local services.
> - CI tears it down.
> - Confidence: SDK behaviors verified end-to-end without an AWS account.

## Pricing

- **Community edition: free.**
- **Pro:** ~$30/mo per dev.

## Gotchas

- **Not 100% AWS parity.** Many services lack edge cases. Always pair with at least some real-AWS integration tests.
- **State is per-container.** Reset between test runs.
- **CloudFormation/CDK works** for many resources but some types are unsupported.
- **Performance differs** from real AWS — don't use LocalStack to benchmark.
- **For Lambda, LocalStack runs your code in a local Docker image** — runtime version match is your responsibility.

## Related

- [SAM](../07-devops-iac/sam.md) — `sam local invoke` for Lambda + APIGW locally
- [DynamoDB Local](#) — official mini-emulator (DDB only)
- [Testcontainers](#) — for popping up DBs in tests
