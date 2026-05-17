# DynamoDB

**TL;DR** — Fully managed NoSQL key-value/document DB. Single-digit ms latency at any scale. Serverless, pay-per-request or provisioned. No instances to size.

## What it is

A managed K/V + document store. You define tables with a **partition key** (and optional **sort key**); DynamoDB shards across many backing servers automatically. No instances, no patching, just an API.

## Why it exists

Relational DBs hit scaling walls. DynamoDB solves a narrower problem (K/V lookups, range scans, no joins) and gives you predictable latency at unbounded scale. Powers Amazon.com itself.

## Key concepts

- **Table** — a collection of items. Schemaless except for keys.
- **Item** — a row (max 400 KB).
- **Attribute** — a field on an item.
- **Partition key (PK)** — hashed to find the shard. Sometimes called "hash key."
- **Sort key (SK)** — optional, lets you order/range-query within a partition.
- **Primary key** — PK alone, or PK+SK composite.
- **Item collection** — all items sharing the same PK (have to fit in 10 GB total).
- **GSI (Global Secondary Index)** — alternate PK/SK. Eventually consistent. Replicated table data, billed separately.
- **LSI (Local Secondary Index)** — alternate SK, same PK. Strongly consistent. Must be defined at table create time.
- **Capacity:**
  - **On-Demand** — pay per request, scales instantly.
  - **Provisioned** — set RCUs/WCUs, optional auto-scaling.
- **TTL** — auto-delete items after a timestamp.
- **Streams** — change-capture stream (CDC) → Lambda / Kinesis.
- **Global Tables** — multi-region active-active replication.
- **DAX** — DynamoDB Accelerator: in-memory caching layer.
- **PartiQL** — SQL-ish query language for DynamoDB (limited; native API is preferred).

## Single-table design

The DynamoDB community's signature pattern: model **all entities in one table**, using PK/SK to encode relationships. Faster at scale but harder to design.

Example for ShareDeal-style entities:

| PK | SK | Type | Other attributes |
|---|---|---|---|
| `USER#42` | `PROFILE` | User | name, email |
| `USER#42` | `ORDER#2026-05-18-abc` | Order | total, status |
| `USER#42` | `ORDER#2026-05-18-def` | Order | total, status |
| `PROD#999` | `PROFILE` | Product | name, price |

`Query` on `PK = USER#42, SK begins_with ORDER#` → list a user's orders in one round-trip.

## Real-world example

> Session store for a web app:
> - Table: `sessions`, PK = `sessionId`.
> - On login → `PutItem` with TTL = `now + 8h`.
> - Each request → `GetItem`.
> - DynamoDB auto-deletes expired sessions.
> - 10M sessions, billions of GETs per month, cost ~$50.

## Usage

### Create table

```bash
aws dynamodb create-table \
  --table-name users \
  --attribute-definitions AttributeName=userId,AttributeType=S \
  --key-schema AttributeName=userId,KeyType=HASH \
  --billing-mode PAY_PER_REQUEST
```

### Put / Get / Update / Query

```bash
aws dynamodb put-item --table-name users \
  --item '{"userId":{"S":"u_42"},"name":{"S":"Arif"},"email":{"S":"arif@example.com"}}'

aws dynamodb get-item --table-name users \
  --key '{"userId":{"S":"u_42"}}'

aws dynamodb update-item --table-name users \
  --key '{"userId":{"S":"u_42"}}' \
  --update-expression "SET #n = :v" \
  --expression-attribute-names '{"#n":"name"}' \
  --expression-attribute-values '{":v":{"S":"Arif R."}}'

aws dynamodb query --table-name orders \
  --key-condition-expression "PK = :u AND begins_with(SK, :o)" \
  --expression-attribute-values '{":u":{"S":"USER#42"},":o":{"S":"ORDER#"}}'
```

### Node SDK v3 with DocumentClient (auto marshalling)

```js
import { DynamoDBClient } from "@aws-sdk/client-dynamodb";
import { DynamoDBDocumentClient, PutCommand, GetCommand, QueryCommand } from "@aws-sdk/lib-dynamodb";

const ddb = DynamoDBDocumentClient.from(new DynamoDBClient({ region: "ap-south-1" }));

await ddb.send(new PutCommand({
  TableName: "users",
  Item: { userId: "u_42", name: "Arif", email: "arif@example.com" },
}));

const { Item } = await ddb.send(new GetCommand({
  TableName: "users",
  Key: { userId: "u_42" },
}));
```

### Transactions

Up to 100 items across multiple tables in one ACID transaction:

```js
await ddb.send(new TransactWriteCommand({
  TransactItems: [
    { Update: { TableName: "accounts", Key: { id: "a1" }, UpdateExpression: "SET bal = bal - :v", ExpressionAttributeValues: { ":v": 100 }, ConditionExpression: "bal >= :v" } },
    { Update: { TableName: "accounts", Key: { id: "a2" }, UpdateExpression: "SET bal = bal + :v", ExpressionAttributeValues: { ":v": 100 } } },
  ],
}));
```

### Streams + Lambda (CDC)

Enable streams → wire to Lambda → react to writes (audit logs, materialized views, search indexing).

## Pricing

### On-Demand
- **Writes:** $1.25 per million.
- **Reads:** $0.25 per million (strongly consistent) / $0.125 (eventually consistent).
- **Storage:** $0.25/GB-mo.

### Provisioned
- **WCU:** $0.00065/hr (1 WCU = 1 write/s of 1 KB).
- **RCU:** $0.00013/hr (1 RCU = 1 strongly consistent read/s of 4 KB, or 2 eventually consistent).
- Auto-scaling supported. Reserved capacity available.

## When to use DynamoDB

- Need < 10 ms latency at scale.
- Workload is mostly K/V or limited-pattern queries.
- Need to grow from 0 → millions of QPS without re-architecting.
- Building serverless apps (DynamoDB + Lambda + API Gateway).

## When NOT to use DynamoDB

- Need ad-hoc SQL / joins / `GROUP BY`.
- Complex reporting → use Aurora + Athena instead.
- Items > 400 KB.
- You only have a tiny app (Postgres on Aurora Serverless is friendlier).

## Gotchas

- **Hot partitions** — bad PK choice = 1 partition gets hammered, throttling.
- **Item size limit 400 KB.** Store big blobs in S3, keep pointers in DDB.
- **No native joins.** Single-table design or multiple queries.
- **Eventually consistent reads** by default — cheaper but ~1 second lag possible.
- **Provisioned mode requires forecasting.** Auto-scaling helps; On-Demand removes guessing entirely.
- **GSI writes cost extra WCUs** — every write replicates to every GSI.
- **Strong consistency is 2x cost** of eventual.
- **Sparse GSIs** are powerful but easy to misunderstand.
- **Scan is expensive** — full table read. Use `Query` whenever possible.

## Related

- [Streams + Lambda](../06-messaging-integration/eventbridge.md) — CDC patterns
- [DAX](#) — in-memory cache for DynamoDB
- [Aurora](./aurora.md) — relational alternative
- [MemoryDB](./memorydb.md) — if you want Redis-style API with DDB-like durability
