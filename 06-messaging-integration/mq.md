# Amazon MQ

**TL;DR** — Managed ActiveMQ and RabbitMQ. For when you have legacy apps using JMS/AMQP/MQTT/STOMP and don't want to refactor to SQS/SNS.

## What it is

Managed message broker. AWS runs the broker, you bring queues/topics/exchanges. Two engines:
- **ActiveMQ** (Classic and Artemis) — JMS, AMQP 1.0, MQTT, OpenWire, STOMP.
- **RabbitMQ** — AMQP 0-9-1, MQTT.

## Why pick MQ vs SQS/SNS

- **You have existing apps using JMS/AMQP/MQTT/STOMP** — drop-in.
- **You need broker-side routing primitives** (RabbitMQ exchanges, message TTLs, priorities, dead-lettering chains).
- **You need broker-side message ordering** with consumer groups.

Otherwise: prefer SQS/SNS/EventBridge for AWS-native simplicity.

## Real-world example

> Insurance company has 15 years of Java apps using ActiveMQ. They lift-and-shift to AWS:
> - Spin up Amazon MQ ActiveMQ (single-instance or active-standby).
> - Update broker URL in apps.
> - Done — no code change.

## Usage

```bash
aws mq create-broker \
  --broker-name sd-mq \
  --engine-type RABBITMQ --engine-version 3.13 \
  --host-instance-type mq.m5.large \
  --deployment-mode CLUSTER_MULTI_AZ \
  --publicly-accessible \
  --users '[{"Username":"admin","Password":"changeme"}]'
```

## Pricing

- Pay per broker-hour ($0.30+/hr) + EBS.
- Cluster mode multiplies.

## Gotchas

- **Pricier than SQS** by a lot at scale.
- **Less elastic** — you size the broker.
- **AMQP/JMS/etc. semantics** — pick the engine that matches your client.

## Related

- [SQS](./sqs.md)
- [MSK](./msk.md)
