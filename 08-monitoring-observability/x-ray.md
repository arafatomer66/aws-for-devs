# X-Ray (and Application Signals)

**TL;DR** — Distributed tracing. See request paths across Lambda → API GW → DynamoDB → SQS → another Lambda. Auto-instrumented for many AWS SDKs.

## What it is

X-Ray collects traces (a chain of "segments" per service hop) and renders a **service map** plus per-request trace timelines. Helps you find latency bottlenecks and error origins in distributed systems.

**Application Signals** (newer, built on X-Ray) gives you OpenTelemetry-compatible traces + SLOs + auto-generated dashboards.

## Key concepts

- **Trace** — one end-to-end request.
- **Segment** — work done by one service.
- **Subsegment** — work within a segment (DB call, downstream HTTP).
- **Sampling rule** — how many requests to trace (default 1 req/s + 5%).
- **Service map** — topology view derived from traces.
- **Annotations** — indexed key/value pairs (queryable).
- **Metadata** — non-indexed extra data.

## Real-world example

> ShareDeal `/checkout` endpoint is slow:
> - X-Ray service map shows `/checkout` → `payment-svc` → `RDS query` is the bottleneck.
> - Trace timeline: 90% of 800 ms latency is the DB call.
> - Add an index → latency drops to 80 ms.

## Usage

### Enable on Lambda

```bash
aws lambda update-function-configuration --function-name charge \
  --tracing-config Mode=Active
```

Add IAM `AWSXRayDaemonWriteAccess` to the role.

### Enable on API Gateway

Console → Stage → Enable X-Ray Tracing.

### Enable on ECS task

Run the X-Ray daemon as a sidecar, or use Fargate's built-in support; add `AWSXRayDaemonWriteAccess` to task role.

### Auto-instrumentation

For Lambda + AWS SDK v3 (Node):
```js
import { captureAWSv3Client } from "aws-xray-sdk-core";
import { DynamoDBClient } from "@aws-sdk/client-dynamodb";
const ddb = captureAWSv3Client(new DynamoDBClient({}));
```

Or use **AWS Distro for OpenTelemetry (ADOT)** — recommended for new projects. Standard OTel SDK exports to X-Ray (or other backends).

### Annotations

```js
import AWSXRay from "aws-xray-sdk-core";
const seg = AWSXRay.getSegment();
seg.addAnnotation("userId", userId);
seg.addAnnotation("orderTotal", total);
```

## Pricing

- **First 100,000 traces/mo: free.**
- **Beyond:** $5.00 per million traces.
- **Trace retrieval:** $0.50 per million scanned.

Cheap for most apps; Application Signals can cost more.

## OpenTelemetry vs X-Ray

- X-Ray SDK is **AWS-proprietary** (still supported).
- **ADOT** (AWS Distro for OpenTelemetry) is the modern path — OTel SDK, export to X-Ray + Prometheus + Honeycomb + whatever.
- Application Signals natively understands OTel.

## Gotchas

- **Sampling rules matter at scale.** Tracing 100% can be expensive and high-overhead.
- **Cross-account traces need explicit setup.**
- **30-day retention** for trace data.
- **Lambda cold start adds a segment** — useful but can clutter analysis.

## Related

- [CloudWatch](./cloudwatch.md)
- [Application Signals](#)
- ADOT — AWS-managed OpenTelemetry Collector
