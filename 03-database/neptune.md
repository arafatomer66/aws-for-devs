# Neptune

**TL;DR** — Managed graph database. Supports Gremlin (property graph), openCypher, and SPARQL (RDF). For social networks, recommendations, fraud detection, knowledge graphs.

## What it is

A graph-native managed DB. Stores nodes/edges/properties and answers traversal queries like "friends of friends" or "shortest path from A to B" or "all transactions linked to this account."

## Key concepts

- **Cluster** — primary + up to 15 replicas + shared storage (Aurora-style).
- **Query languages:**
  - **Gremlin** — graph traversal, imperative.
  - **openCypher** — Neo4j-style declarative.
  - **SPARQL** — for RDF/triple stores.
- **Loader** — bulk-load from S3 (CSV/RDF formats).
- **Streams** — change-feed.
- **Neptune Analytics** — newer, in-memory graph analytics variant for queries on whole graphs.
- **Neptune ML** — built-in graph neural network features.

## Real-world example

> Fraud detection: "find all accounts within 3 hops of a known fraudulent account, sharing devices or IPs."
>
> ```cypher
> MATCH (f:Fraud {id: 'x1'})-[*1..3]-(suspect:Account)
> WHERE NOT suspect.flagged
> RETURN suspect.id
> ```

## Usage

### Create cluster

Just like Aurora — use console / CDK / CloudFormation. Endpoint is `xxxx.cluster-xxxx.<region>.neptune.amazonaws.com:8182`.

### Load data

```bash
curl -X POST -H 'Content-Type: application/json' \
  https://my-cluster:8182/loader -d '{
    "source": "s3://my-bucket/graph-data/",
    "format": "csv",
    "iamRoleArn": "arn:aws:iam::123456789012:role/NeptuneLoadFromS3",
    "region": "ap-south-1"
  }'
```

### Query (Gremlin)

```python
from gremlin_python.driver.driver_remote_connection import DriverRemoteConnection
from gremlin_python.process.anonymous_traversal import traversal

g = traversal().with_remote(DriverRemoteConnection("wss://...:8182/gremlin", "g"))
print(g.V().has("Account", "id", "x1").out("knows").limit(10).valueMap().toList())
```

## Pricing

- Instance ($0.30/hr+) + storage ($0.10/GB-mo) + I/O.
- Similar to Aurora pricing.

## When to use a graph DB

- Highly connected data (social, supply chain, identity graphs).
- Traversals more important than aggregations.
- Pattern matching across relationships.

When NOT:
- Simple K/V access → Dynamo.
- Tabular analytics → Redshift.
- Embed graph in JSON in Postgres if your graph is small.

## Gotchas

- **Three different query languages** — pick one, stick with it.
- **No SQL** — your devs need to learn Gremlin/Cypher/SPARQL.
- **Cold starts after long idle** — Neptune Serverless helps.
- **Bulk load is the fast path** — single-vertex writes are slow.

## Related

- [Neptune ML](#)
- [DynamoDB](./dynamodb.md) — when you don't really need a graph
