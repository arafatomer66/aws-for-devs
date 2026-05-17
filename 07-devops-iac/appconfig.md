# AppConfig

**TL;DR** — Managed feature flags and dynamic config. Validates, deploys gradually, monitors with alarms, rolls back on failure. Sub-service of Systems Manager.

## What it is

A config-as-data delivery system with **gradual rollouts** and **automatic rollback**. Your app polls / streams configs; AppConfig serves a versioned value with deployment rules.

## Key concepts

- **Application** — logical grouping (e.g., `ShareDeal-API`).
- **Environment** — dev / staging / prod.
- **Configuration profile** — schema + source (S3, Parameter Store, Secrets Manager, AppConfig hosted, Code Commit).
- **Deployment strategy** — Linear / Exponential / All-at-once, bake time, growth rate.
- **Validator** — JSON Schema or Lambda; gates bad configs.
- **Alarm rollback** — if a CloudWatch alarm fires during deploy, roll back automatically.
- **Feature Flags** — purpose-built profile type for flags.

## Real-world example

> ShareDeal feature flag rollout:
> - Flag `new_checkout` defaults off.
> - Toggle to on in staging → bake → prod.
> - Deployment strategy: 10% / 5 min / 10 min bake.
> - CloudWatch alarm on `5xx > 1%` → AppConfig auto-rolls back the deploy.
> - App reads flag every 60 s via the AppConfig Agent.

## Usage

### Create application + environment + flag

Easiest via console. Or:

```bash
aws appconfig create-application --name sharedeal-api

aws appconfig create-environment --application-id <app-id> --name prod \
  --monitors '[{"AlarmArn":"arn:aws:cloudwatch:..:alarm:5xx-high"}]'

aws appconfig create-configuration-profile --application-id <app-id> \
  --name feature-flags --type "AWS.AppConfig.FeatureFlags" --location-uri hosted
```

Then publish flag values and start a deployment.

### Read from a Lambda or ECS

Use the **AppConfig Lambda extension** or the **agent for ECS/EC2** — they cache the config locally and refresh in the background. Your app reads from `localhost:2772`:

```bash
curl http://localhost:2772/applications/sharedeal-api/environments/prod/configurations/feature-flags
```

In Node:
```js
const res = await fetch("http://localhost:2772/applications/sharedeal-api/environments/prod/configurations/feature-flags");
const flags = await res.json();
if (flags.new_checkout?.enabled) { /* new path */ }
```

## Pricing

- **Per request:** $0.0008 per config request (when fetching directly).
- **Per active config:** $0.0008 per receiver per hour (when using agent/extension's "active" mode).

Both are tiny for typical apps. Free tier covers small projects.

## AppConfig vs LaunchDarkly / Split / Unleash

- **AppConfig** — AWS-native, cheap, basic feature flags.
- **LaunchDarkly / Split** — Best-in-class flag features (targeting, experiments, audit), pricier.
- **Unleash / Flagsmith** — OSS alternatives, self-host.

If you need user-level targeting + experiments, a dedicated flag vendor is better. For basic on/off + gradual rollout, AppConfig is fine.

## Gotchas

- **Agent / extension caching** means changes take up to `polling_interval` seconds to propagate.
- **Validator failures abort deployment** — make sure your JSON Schema matches reality.
- **Alarm rollback** only triggers during the deploy window — keep alarms tuned.
- **No native user-attribute targeting** — for "show feature to 10% of users in BD", you target by sub-key in the config and your app does the matching.

## Related

- [Systems Manager](./systems-manager.md)
- [Parameter Store](../05-security-iam/parameter-store.md)
- [CloudWatch](../08-monitoring-observability/cloudwatch.md)
