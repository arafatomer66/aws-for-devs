# Lambda — Serverless Functions

**TL;DR** — Upload a function, AWS runs it on demand, you pay per ms × memory. No servers to manage. 15-minute max execution.

## What it is

You give Lambda:
- A zip of your code (or a container image), and
- A handler function name (e.g. `index.handler`).

Lambda runs that function whenever something **invokes** it — HTTP request, S3 upload, SQS message, cron, etc. Zero infrastructure to manage.

## Why it exists

Most code is idle most of the time. Why run a server 24/7 for a webhook that fires 100 times a day? Lambda bills you only for the time your code is executing (down to the millisecond).

## Key concepts

- **Function** — your code + config.
- **Handler** — the entry point function name.
- **Runtime** — Node.js, Python, Java, Go, Ruby, .NET, custom. Each version is a Lambda "runtime."
- **Trigger / event source** — what invokes the function (API Gateway, S3, SQS, EventBridge, Cron, etc.).
- **Cold start** — first invocation requires spinning up a container (~100–500 ms for Node, longer for JVM). Subsequent calls reuse it.
- **Warm container** — kept around for ~5–15 mins.
- **Concurrency** — how many executions can run in parallel. Default account-wide soft limit: 1000.
- **Reserved concurrency** — guarantee N slots to one function.
- **Provisioned concurrency** — pre-warm N execution environments — no cold starts (costs extra).
- **Layer** — shared zip of dependencies (max 5 layers per function).
- **Destination** — async result routing to SQS/SNS/EventBridge/Lambda.
- **Lambda@Edge** — Lambda that runs at CloudFront edge locations.

## Limits (worth memorizing)

- **Memory:** 128 MB → 10,240 MB (10 GB). CPU scales linearly with memory.
- **Timeout:** 15 minutes max.
- **Deployment package:** 50 MB zipped (250 MB unzipped). Or **10 GB container image**.
- **Temp storage (`/tmp`):** 512 MB by default, configurable up to 10 GB.
- **Payload:** 6 MB sync invoke, 256 KB async invoke, 6 MB response.
- **Concurrent executions per account/region:** 1000 (soft, raise via support).

## Real-world example

> Image upload pipeline:
>
> 1. User uploads JPG to S3 bucket `uploads/`.
> 2. S3 fires event → Lambda `resize-image` triggered.
> 3. Lambda downloads image, generates 3 thumbnails, uploads to `thumbnails/`.
> 4. Lambda writes a row to DynamoDB with thumbnail URLs.
>
> Cost at 1M uploads/month: ~$0.20 in Lambda. Zero servers.

## Usage

### Create a function (Node.js)

`index.mjs`:
```js
export const handler = async (event) => {
  console.log("Event:", JSON.stringify(event));
  return {
    statusCode: 200,
    body: JSON.stringify({ message: "hello from lambda" }),
  };
};
```

```bash
zip function.zip index.mjs
aws lambda create-function \
  --function-name hello-lambda \
  --runtime nodejs20.x \
  --role arn:aws:iam::123456789012:role/lambda-basic \
  --handler index.handler \
  --zip-file fileb://function.zip
```

### Invoke

```bash
aws lambda invoke --function-name hello-lambda --payload '{}' out.json
cat out.json
```

### Update code

```bash
zip function.zip index.mjs
aws lambda update-function-code --function-name hello-lambda --zip-file fileb://function.zip
```

### Container image deploy

```Dockerfile
FROM public.ecr.aws/lambda/nodejs:20
COPY index.mjs ${LAMBDA_TASK_ROOT}
CMD ["index.handler"]
```

Push to ECR, then `--package-type Image --code ImageUri=...`.

### CDK example

```ts
new lambda.Function(this, 'ResizeImage', {
  runtime: lambda.Runtime.NODEJS_20_X,
  handler: 'index.handler',
  code: lambda.Code.fromAsset('./src'),
  memorySize: 512,
  timeout: cdk.Duration.seconds(30),
  environment: { BUCKET: thumbBucket.bucketName },
});
```

## Common triggers

| Trigger | Use case |
|---|---|
| API Gateway | REST/HTTP APIs |
| Function URL | Public HTTPS without API Gateway |
| S3 | On object upload/delete |
| SQS | Process queue messages |
| SNS | Pub/sub fan-out |
| EventBridge | Scheduled (cron) or event-driven |
| DynamoDB Streams | React to table changes |
| Kinesis | Stream processing |
| Cognito | Pre-signup hooks, custom auth |
| ALB | HTTP via Application Load Balancer |
| Step Functions | Workflow tasks |

## Pricing model

- **Requests:** $0.20 per 1M.
- **Compute:** $0.0000166667 per GB-second.
- **Free tier (forever):** 1M requests + 400k GB-seconds/mo.

Example: 1M invocations at 200 ms, 512 MB = 1M × 0.2 × 0.5 GB = 100k GB-s = **$1.67 + $0.20 = ~$1.87**.

Always-free for most hobby projects.

## Cold starts — how to handle

- **Provisioned Concurrency** — pre-warm N instances. Costs extra (~$10/mo per warm slot).
- **Use a fast runtime** — Node and Python cold start in ~100ms. JVM is 500ms–2s; use **SnapStart** for Java.
- **Smaller package** — less code to load.
- **Avoid initializing in handler** — connect to DB *outside* the handler so it's reused.

```js
// Init once per container (good)
const db = new Pool({ ... });

export const handler = async (event) => {
  const result = await db.query("SELECT 1");
  return result;
};
```

## Versions & aliases

- Each `update-function-code` is a new **version** (immutable).
- An **alias** like `prod` points to a version. You can shift traffic gradually (canary, 10% → 100%).

```bash
aws lambda publish-version --function-name hello-lambda
aws lambda create-alias --function-name hello-lambda --name prod --function-version 3
```

## Gotchas

- **15-minute timeout.** For longer work use Step Functions, ECS, or Batch.
- **Cold starts for VPC-bound Lambdas** improved hugely in 2019 (Hyperplane ENIs), but JVM/Spring Boot still hurts.
- **No state between invocations.** Use DynamoDB / S3 / ElastiCache for state.
- **No persistent disk.** `/tmp` is up to 10 GB, but wiped frequently.
- **Concurrency throttling** — if you hit 1000, new invocations get 429s. Buffer with SQS.
- **Synchronous invokes that fail are not retried.** Async (SNS, S3 events) auto-retry 2x by default — configure a DLQ.
- **Recursive invokes will burn money fast** — a Lambda writing to S3 that triggers itself causes infinite loops. AWS now auto-stops these but watch out.
- **Container images are not cheaper or faster** for small functions. Zip is usually fine.

## Related

- [API Gateway](../04-networking/api-gateway.md) — front Lambda with HTTP
- [SQS](../06-messaging-integration/sqs.md) — buffer Lambda
- [EventBridge](../06-messaging-integration/eventbridge.md) — event-driven Lambda
- [Step Functions](../06-messaging-integration/step-functions.md) — orchestrate Lambdas
- [Fargate](./fargate.md) — when 15 min isn't enough
