# DocumentDB

**TL;DR** — MongoDB-API-compatible managed database. Same drivers, AWS storage layer like Aurora. Not 100% MongoDB feature parity.

## What it is

A managed document database that speaks MongoDB's wire protocol (compatibility tested up to MongoDB 5.0 at the time of writing). Built on AWS's shared distributed storage layer (same tech as Aurora) — 6-way replicated across 3 AZs.

## Why it exists

MongoDB Atlas works, but if you're all-in on AWS and want one-vendor billing, IAM integration, VPC native, etc., DocumentDB fits. **It's not MongoDB** — it's AWS's reimplementation.

## Key concepts

- **Cluster** — primary + up to 15 replicas + shared storage.
- **Instance class** — `db.r6g.large` etc., similar to RDS.
- **Endpoints** — cluster (writer) + reader.
- **Backups** — continuous, PITR.
- **Global clusters** — cross-region.
- **Elastic Clusters** — sharded, auto-scaling variant (newer).

## Compatibility caveats

- Most MongoDB drivers + queries work.
- **Missing or limited:** some aggregation operators, transactions across shards (until Elastic Clusters), text search (use OpenSearch), TTL on capped collections, change streams subtle differences.
- Always test your specific app against DocumentDB before committing.

## Real-world example

> A team has an existing Mongoose app and doesn't want to manage Mongo replica sets. They:
> - Migrate dump → DocumentDB.
> - App connection string updated.
> - Most queries Just Work; they replace 2 unsupported aggregation stages.

## Usage

### Create cluster

```bash
aws docdb create-db-cluster \
  --db-cluster-identifier docs-cluster \
  --engine docdb \
  --master-username admin --master-user-password 'change-me!' \
  --vpc-security-group-ids sg-0123 --db-subnet-group-name docdb-subnets

aws docdb create-db-instance \
  --db-instance-identifier docs-writer \
  --db-cluster-identifier docs-cluster \
  --instance-class db.r6g.large --engine docdb
```

### Connect (Node Mongoose)

```js
import mongoose from "mongoose";
await mongoose.connect("mongodb://admin:***@docs-cluster.cluster-xyz.docdb.amazonaws.com:27017/mydb?tls=true&replicaSet=rs0&readPreference=secondaryPreferred&retryWrites=false");
```

Note: `retryWrites=false` is required.

TLS uses the rds-combined-ca-bundle.

## Pricing

- Instance + storage + I/O (or I/O-optimized mode).
- Similar to Aurora pricing, slightly different rates.
- A small `db.r6g.large` cluster ≈ $200-400/mo.

## When DocumentDB vs Mongo Atlas vs DynamoDB

- **DocumentDB** — you have existing MongoDB code, want AWS-native.
- **MongoDB Atlas** — you want full Mongo features, vendor flexibility, multi-cloud.
- **DynamoDB** — starting fresh, want managed scaling + lower ops, OK with K/V model.

## Gotchas

- **Not full MongoDB compatibility.** Check the docs page on supported operators.
- **`retryWrites=false`** in connection string.
- **Cluster endpoint always writes** — can't write through reader endpoint.
- **No global secondary index concept** like DynamoDB — Mongo-style indexes.
- **No JSON Schema validation** of the same shape as MongoDB.

## Related

- [DynamoDB](./dynamodb.md)
- [Aurora](./aurora.md) — same underlying storage layer
