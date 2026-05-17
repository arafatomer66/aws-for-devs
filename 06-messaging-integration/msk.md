# MSK — Managed Streaming for Apache Kafka

**TL;DR** — Managed Kafka. AWS runs brokers + ZooKeeper/KRaft, you bring topics + producers + consumers. Use if you already speak Kafka. MSK Serverless and Express Brokers exist for cheaper/easier modes.

## What it is

Apache Kafka, managed. AWS provisions brokers, applies patches, handles broker failure, snapshots, monitoring. You connect with standard Kafka clients.

Newer modes:
- **MSK Serverless** — autoscaling, no broker sizing. Pay per use.
- **Express Brokers** — newer brokers with faster scaling and better economics (2024+).

## Key concepts

- **Cluster** — set of brokers.
- **Topic** — Kafka topic.
- **Partition** — Kafka partition.
- **Authentication:** TLS, SASL/SCRAM, IAM.
- **MSK Connect** — managed Kafka Connect (S3, OpenSearch, JDBC sinks/sources).
- **Schema Registry** — Glue Schema Registry works with MSK.

## Real-world example

> An enterprise migrating from on-prem Kafka:
> - Spin up MSK cluster with 3 brokers across 3 AZs.
> - Existing producers/consumers point at MSK bootstrap servers (with TLS).
> - Use IAM auth so service roles authenticate to topics without static creds.
> - MSK Connect ships data to S3 in Parquet for analytics.

## Usage

### Create (Provisioned)

```bash
aws kafka create-cluster --cluster-name sd-kafka \
  --kafka-version 3.7.x \
  --number-of-broker-nodes 3 \
  --broker-node-group-info '{
    "InstanceType":"kafka.m7g.large",
    "ClientSubnets":["subnet-a","subnet-b","subnet-c"],
    "SecurityGroups":["sg-msk"],
    "StorageInfo":{"EBSStorageInfo":{"VolumeSize":100}}
  }' \
  --encryption-info '{"EncryptionInTransit":{"InCluster":true,"ClientBroker":"TLS"}}' \
  --client-authentication '{"Sasl":{"Iam":{"Enabled":true}}}'
```

### Get bootstrap servers

```bash
aws kafka get-bootstrap-brokers --cluster-arn arn:aws:kafka:..:cluster/sd-kafka/...
```

### Produce (using kafka-console-producer + IAM auth)

Add `aws-msk-iam-auth` to client classpath; configure `security.protocol=SASL_SSL`, `sasl.mechanism=AWS_MSK_IAM`.

### MSK Serverless

```bash
aws kafka create-cluster-v2 --cluster-name sd-kafka-serverless \
  --serverless '{
    "VpcConfigs":[{"SubnetIds":["subnet-a","subnet-b","subnet-c"],"SecurityGroupIds":["sg-msk"]}],
    "ClientAuthentication":{"Sasl":{"Iam":{"Enabled":true}}}
  }'
```

## Pricing

- **Provisioned brokers:** ~$0.21/hr per `kafka.m7g.large` + EBS.
  - 3-broker cluster ≈ $450-600/mo before storage.
- **Express:** roughly same; better scaling characteristics.
- **Serverless:** $0.75 per cluster-hour + per partition + per GB.
- **MSK Connect:** ~$0.11/MCU-hr.

## When MSK vs Kinesis Data Streams

- **MSK** — you have existing Kafka ecosystem (Schema Registry, Streams, Connectors), or need at-least-once with custom semantics you don't get in Kinesis.
- **Kinesis** — starting fresh, want lowest ops, deep AWS integration.

## Gotchas

- **More moving parts than Kinesis.** Even managed, Kafka is Kafka.
- **Public access not supported by default** — VPC-private mostly. Public access add-on exists.
- **MSK Connect connectors** — fewer than open-source Kafka Connect plugins.
- **IAM auth** is convenient but Kafka clients need the AWS plugin.

## Related

- [Kinesis](./kinesis.md)
- [Lambda](../01-compute/lambda.md) — Lambda event source for MSK
- [Glue Schema Registry](../10-analytics/glue.md)
