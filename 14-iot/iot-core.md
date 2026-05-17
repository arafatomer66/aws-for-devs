# AWS IoT Core

**TL;DR** — Managed MQTT broker + device registry + rules engine + device shadow. Connect millions of devices over MQTT/HTTPS/WebSocket, route messages into AWS.

## What it is

The hub of AWS's IoT stack. Provides:
- **MQTT broker** (and MQTT-over-WSS, HTTPS, LoRaWAN).
- **Device registry** ("things") with certs/policies.
- **Rules engine** — SQL on MQTT messages, routes to Lambda / S3 / DynamoDB / Kinesis / Timestream / etc.
- **Device Shadow** — JSON state synced between device + cloud, even when device offline.
- **Jobs** — fleet OTA updates / commands.
- **Defender** — anomaly detection on device behavior.

## Key concepts

- **Thing** — a registered device.
- **Certificate** — X.509 cert per device for mTLS.
- **Thing policy** — IoT-specific IAM-like policy (what topics it can publish/subscribe to).
- **Topic** — MQTT topic string (`sd/devices/{deviceId}/telemetry`).
- **Rule** — SQL: `SELECT * FROM 'sd/devices/+/telemetry' WHERE temp > 80`.
- **Shadow** — `$aws/things/<name>/shadow/...` topics; reported vs desired state.
- **Fleet provisioning** — bootstrap devices at scale (claim certs).

## Real-world example

> Smart-meter fleet:
> - 50,000 devices, each publishes consumption every minute to `meters/{id}/usage`.
> - IoT Rule: `SELECT id, kwh FROM 'meters/+/usage'` → Kinesis Data Firehose → S3 (Parquet) → Athena.
> - Per-device shadow tracks last-known firmware version.
> - IoT Jobs roll out firmware updates in waves.

## Usage

### Create a thing + cert

```bash
aws iot create-thing --thing-name meter-001
aws iot create-keys-and-certificate --set-as-active \
  --certificate-pem-outfile meter-001.cert.pem \
  --public-key-outfile meter-001.public.key \
  --private-key-outfile meter-001.private.key
# Returns certificateArn — attach a policy + the thing
```

### Attach policy

```json
{
  "Version":"2012-10-17",
  "Statement":[
    { "Effect":"Allow","Action":"iot:Connect","Resource":"arn:aws:iot:..:client/${iot:Connection.Thing.ThingName}" },
    { "Effect":"Allow","Action":"iot:Publish","Resource":"arn:aws:iot:..:topic/meters/${iot:Connection.Thing.ThingName}/*" }
  ]
}
```

### Rule example

```sql
SELECT id, temperature, timestamp() AS ts
FROM 'sensors/+/data'
WHERE temperature > 80
```
Action: invoke Lambda `alert-on-overheat`.

### Device-side (Node MQTT)

```js
import { mqtt, iot } from "aws-iot-device-sdk-v2";

const client = new mqtt.MqttClientConnection(/* configured with cert + key + endpoint */);
await client.connect();
await client.publish("meters/001/usage", JSON.stringify({ kwh: 0.42 }), mqtt.QoS.AtLeastOnce);
```

### Endpoint

`<account-id>.iot.<region>.amazonaws.com` (port 8883 mTLS / 443 ALPN / 443 WSS).

## Pricing

- **Connectivity:** $0.08 per million connection-minutes.
- **Messages:** $1.00 per million.
- **Registry:** $1.25 per million ops.
- **Shadow/Jobs/Rules:** small per-op fees.
- **Defender, Device Management:** separate per-device fees.

A 100k-device fleet at 1 msg/min ≈ a few hundred $/mo.

## Related AWS IoT services

- **IoT Greengrass** — edge runtime; run Lambda + ML locally on devices.
- **IoT SiteWise** — industrial telemetry + asset hierarchy.
- **IoT Events** — detector models (state machines on device events).
- **IoT FleetWise** — vehicle data.
- **IoT TwinMaker** — digital twins.

## Gotchas

- **mTLS certs everywhere** — manage cert lifecycle (rotation, revocation).
- **Topic structure design** matters — plan `account/site/device/datatype` early.
- **Quota: messages per connection per sec** — verify your fleet density.
- **MQTT broker has limits per region** — check before scaling to millions.
- **MQTT 5 supported** as of 2024 — use it for new fleets.

## Related

- [Kinesis](../06-messaging-integration/kinesis.md)
- [Timestream](../03-database/timestream.md) — time-series store for telemetry
- [Lambda](../01-compute/lambda.md)
