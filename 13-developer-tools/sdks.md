# AWS SDKs

**TL;DR** — Official libraries to call AWS APIs from your app. JS/TS, Python (boto3), Java, Go, Rust, .NET, PHP, Ruby, Swift, Kotlin, C++. Auto-handles signing, retries, pagination.

## What's available

- **JavaScript/TypeScript v3** — `@aws-sdk/client-<service>`, modular.
- **Python: boto3** — the one Python SDK.
- **Java v2** — `software.amazon.awssdk:<service>`, async-friendly.
- **Go v2** — `github.com/aws/aws-sdk-go-v2/...`.
- **Rust** — `aws-sdk-<service>`, official, async.
- **.NET** — `AWSSDK.<Service>` NuGet packages.
- **PHP, Ruby, Swift, Kotlin, C++** — all official.

## Credential resolution (same chain in every SDK)

Each SDK auto-resolves creds in this order:
1. **Hard-coded** in code (don't do this).
2. **Environment variables** — `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_SESSION_TOKEN`.
3. **AWS_PROFILE / `~/.aws/credentials`**.
4. **SSO session.**
5. **Container creds** (ECS task role).
6. **Instance metadata** (EC2 instance profile).
7. **EKS pod identity / IRSA.**

In code, you almost never pass creds — let the chain work.

## Real-world example

### Node.js v3 (modular)

```js
import { DynamoDBClient } from "@aws-sdk/client-dynamodb";
import { DynamoDBDocumentClient, PutCommand } from "@aws-sdk/lib-dynamodb";

const ddb = DynamoDBDocumentClient.from(new DynamoDBClient({ region: "ap-south-1" }));
await ddb.send(new PutCommand({
  TableName: "users",
  Item: { userId: "u_42", name: "Arif" },
}));
```

### Python (boto3)

```python
import boto3
ddb = boto3.resource("dynamodb", region_name="ap-south-1")
table = ddb.Table("users")
table.put_item(Item={"userId":"u_42","name":"Arif"})
```

### Java v2

```java
DynamoDbClient ddb = DynamoDbClient.builder().region(Region.AP_SOUTH_1).build();
ddb.putItem(PutItemRequest.builder()
    .tableName("users")
    .item(Map.of("userId", AttributeValue.fromS("u_42"),
                 "name",   AttributeValue.fromS("Arif")))
    .build());
```

### Go v2

```go
cfg, _ := config.LoadDefaultConfig(context.TODO(), config.WithRegion("ap-south-1"))
ddb := dynamodb.NewFromConfig(cfg)
_, err := ddb.PutItem(ctx, &dynamodb.PutItemInput{
    TableName: aws.String("users"),
    Item: map[string]types.AttributeValue{
        "userId": &types.AttributeValueMemberS{Value: "u_42"},
        "name":   &types.AttributeValueMemberS{Value: "Arif"},
    },
})
```

### Rust

```rust
let config = aws_config::load_from_env().await;
let ddb = aws_sdk_dynamodb::Client::new(&config);
ddb.put_item()
    .table_name("users")
    .item("userId", AttributeValue::S("u_42".into()))
    .item("name",   AttributeValue::S("Arif".into()))
    .send().await?;
```

## Retries and timeouts

All SDKs have built-in **adaptive retries** with exponential backoff + jitter. Defaults:
- **Max attempts:** 3 (or 4 in some SDKs).
- **Retry mode:** `standard` or `adaptive`.

Customize via env (`AWS_RETRY_MODE`, `AWS_MAX_ATTEMPTS`) or in code.

For timeouts, configure HTTP client. Defaults are too generous for serverless — set ~10 s.

## Pagination helpers

```js
// JS v3 paginators
import { paginateListObjectsV2 } from "@aws-sdk/client-s3";
for await (const page of paginateListObjectsV2({ client: s3 }, { Bucket: "my-bucket" })) {
  for (const obj of page.Contents ?? []) console.log(obj.Key);
}
```

```python
# boto3
paginator = s3.get_paginator("list_objects_v2")
for page in paginator.paginate(Bucket="my-bucket"):
    for obj in page.get("Contents", []):
        print(obj["Key"])
```

## SDK observability

- **X-Ray tracing** — wrap clients with `captureAWSv3Client` (JS) / `XRayPatch` (Py) / OTel.
- **CloudWatch metrics** — `AWS/SDK` metrics for retries, errors (newer feature).

## Pricing

- **SDKs are free.** You pay for the API calls' effects.

## Gotchas

- **Long-lived connections matter for Lambda** — keep clients module-scoped, not per-handler.
- **SDK v1 (JS, Java)** is unsupported. Migrate to v3 / v2.
- **JS SDK v3 is modular** — install only `@aws-sdk/client-<svc>` you use to keep bundle small.
- **Boto3 with assume-role** — use `STS AssumeRole` + temp creds, not hardcoded keys.
- **Region pinning** — every client needs a region; check what your code defaults to.

## Related

- [CLI](./cli.md)
- [CDK](../07-devops-iac/cdk.md) — uses SDK constructs under the hood
- [IAM](../05-security-iam/iam.md)
