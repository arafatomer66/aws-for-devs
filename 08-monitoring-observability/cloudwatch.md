# CloudWatch

**TL;DR** — AWS's observability service: metrics, logs, alarms, dashboards, synthetic checks, real-user monitoring, ServiceLens. The default place for AWS-native monitoring.

## What it is

A bundle of services under the CloudWatch umbrella:
- **Metrics** — numeric time-series.
- **Logs** — log streams.
- **Alarms** — fire on metric thresholds.
- **Dashboards** — visualizations.
- **Events / EventBridge** — now its own service.
- **Synthetics** — canary scripts.
- **RUM** — real user monitoring (browser).
- **ServiceLens / Application Signals** — traces + metrics correlation.
- **Container Insights** — ECS/EKS deep metrics.
- **Lambda Insights** — Lambda runtime metrics.

## Key concepts

### Metrics
- **Namespace** — e.g., `AWS/EC2`, `MyApp`.
- **Metric name + dimensions** — `CPUUtilization` with `InstanceId=i-xxx`.
- **Resolution:** 1 min (default) or 1 sec (high-res, more $$).
- **Retention:** 1-sec data 3 hrs, 5-min data 15 days, 1-hr data 15 months.
- **Custom metrics** — your app pushes them via `PutMetricData`.

### Logs
- **Log group** — collection of log streams (one per resource usually).
- **Log stream** — one source of logs (one Lambda execution env, one EC2, etc.).
- **Retention:** default infinite (= expensive). **Set retention on every log group**.
- **Metric filter** — extract metrics from log patterns.
- **Subscription filter** — forward to Kinesis / Firehose / Lambda for processing.
- **Insights** — SQL-like log query language.

### Alarms
- Trigger when metric crosses threshold for N periods.
- Actions: SNS topic, Auto Scaling, EC2 reboot, Lambda.
- **Composite alarms** — boolean combos.

## Real-world example

> ShareDeal monitoring setup:
> - All services log to CloudWatch Logs (`/ecs/api`, `/aws/lambda/charge`, `/var/log/nginx`).
> - Retention 30 days on most, 365 days on audit logs.
> - Custom metrics: `OrdersPlaced`, `PaymentFailures`.
> - Alarms: `5xxRate > 1% for 5 min` → SNS → PagerDuty.
> - Dashboards: per-service tile (CPU, latency, error rate, queue depth).

## Usage

### Send custom metric

```js
import { CloudWatchClient, PutMetricDataCommand } from "@aws-sdk/client-cloudwatch";
const cw = new CloudWatchClient({ region: "ap-south-1" });
await cw.send(new PutMetricDataCommand({
  Namespace: "ShareDeal",
  MetricData: [{
    MetricName: "OrdersPlaced",
    Dimensions: [{ Name: "Env", Value: "prod" }],
    Value: 1,
    Unit: "Count",
  }],
}));
```

For Lambda, use **EMF (Embedded Metric Format)** — just `console.log` a JSON, no API call needed:

```js
console.log(JSON.stringify({
  _aws: {
    Timestamp: Date.now(),
    CloudWatchMetrics: [{
      Namespace: "ShareDeal",
      Dimensions: [["Env"]],
      Metrics: [{ Name: "OrdersPlaced", Unit: "Count" }],
    }],
  },
  Env: "prod",
  OrdersPlaced: 1,
}));
```

### Create an alarm

```bash
aws cloudwatch put-metric-alarm \
  --alarm-name api-5xx-high \
  --metric-name HTTPCode_Target_5XX_Count \
  --namespace AWS/ApplicationELB \
  --statistic Sum --period 60 --evaluation-periods 5 \
  --threshold 10 --comparison-operator GreaterThanThreshold \
  --dimensions Name=LoadBalancer,Value=app/api/abc \
  --alarm-actions arn:aws:sns:ap-south-1:..:oncall
```

### Logs Insights query

```sql
fields @timestamp, @message
| filter @message like /ERROR/
| stats count() by bin(5m)
| sort @timestamp desc
| limit 100
```

### Set log retention (critical for cost)

```bash
aws logs put-retention-policy --log-group-name /ecs/api --retention-in-days 30
```

## Pricing (the expensive bits)

- **Logs ingestion:** $0.50/GB. The #1 way to surprise yourself.
- **Logs storage:** $0.03/GB-mo (compressed).
- **Custom metrics:** $0.30 each per month (first 10k). Charges per dimension combination.
- **API requests:** $0.01 per 1000 (GetMetricData etc.).
- **Alarms:** $0.10 each standard, $0.30 high-res.
- **Synthetics canary runs:** $0.0012 each.
- **Container Insights / Application Signals:** higher tier, can get pricey.

## Cost-control checklist

1. **Set log retention** on every log group (default = infinite = $$$).
2. **Filter logs at source** — `level=INFO` not `level=DEBUG` in prod.
3. **Use EMF** instead of separate PutMetric calls.
4. **Subscription filter to S3 + Athena** if you need long-term log search (much cheaper than CW Logs at scale).
5. **Disable Container Insights** if you don't use it.

## CloudWatch vs Datadog/Honeycomb/etc.

- CloudWatch is good enough for AWS-native monitoring, especially small/medium teams.
- Datadog and friends offer better UX, faster queries, easier correlation, more expensive.
- Many teams use both: CloudWatch for AWS service metrics (free-ish), Datadog for app traces & dashboards.

## Gotchas

- **Cross-account/region dashboards** — possible but fiddly.
- **Logs Insights** is per-region — can't query multiple regions at once.
- **Metric resolution downsamples over time** — old 1-min data becomes 5-min, then 1-hr.
- **PutMetricData has rate limits** — batch when you can.
- **Custom metrics bill per unique dimension combo per minute.** High-cardinality = $$$.

## Related

- [X-Ray](./x-ray.md)
- [CloudTrail](./cloudtrail.md)
- [Application Signals / ServiceLens](#)
- [EventBridge](../06-messaging-integration/eventbridge.md) — alarms feed events
