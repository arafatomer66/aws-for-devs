# OpenSearch

**TL;DR** — Managed Elasticsearch fork. Full-text search, log analytics, vector search. Two flavors: domains (clusters) and Serverless.

## What it is

The OpenSearch Project is a community fork of Elasticsearch 7.10 (when ES went to a non-OSS license). Amazon OpenSearch Service runs it managed.

Two deployment types:
- **Managed domains** — you pick instance types and count.
- **OpenSearch Serverless** — auto-scales, pay per OCU.

## Use cases

- **Full-text search** — product catalogs, doc search.
- **Log analytics** — alternative to CloudWatch Logs Insights for big log volumes.
- **Vector search** — embeddings store for RAG.
- **Security analytics** — security log queries.
- **Observability** — distributed tracing with Trace Analytics.

## Key concepts

- **Domain** — a cluster.
- **Index** — collection of documents (JSON).
- **Shard / replica** — Elasticsearch partitioning.
- **Mapping** — schema (fields + types).
- **Query DSL** — JSON queries.
- **OpenSearch Dashboards** — Kibana fork for visualization.
- **k-NN plugin** — vector search.
- **Fine-grained access control** — internal users + roles.

## Real-world example

> ShareDeal product search:
> - OpenSearch domain (3 nodes, `r6g.large.search`).
> - `products` index with `name`, `description`, `category`, `price`, embedding (`vector`).
> - Live updates via DynamoDB Streams → Lambda → OpenSearch.
> - Query combines BM25 text relevance + vector similarity for "fuzzy semantic" search.

## Usage

### Index a doc

```bash
curl -X POST "https://my-domain.us-east-1.es.amazonaws.com/products/_doc" \
  -H "Content-Type: application/json" \
  -d '{"name":"Mango juice","category":"beverages","price":120}'
```

### Search

```bash
curl -X POST "https://.../products/_search" -d '{
  "query": { "multi_match": { "query": "mango", "fields": ["name^2","description"] } }
}'
```

### Vector kNN

```json
PUT /docs
{
  "mappings": {
    "properties": {
      "embedding": { "type": "knn_vector", "dimension": 768 }
    }
  }
}

POST /docs/_search
{
  "size": 5,
  "query": { "knn": { "embedding": { "vector": [0.1, 0.2, ...], "k": 5 } } }
}
```

## Pricing

- **Domains:** instance + EBS + UltraWarm (cheap warm storage).
- **Serverless:** $0.24 per OCU-hr (indexing + search separate).

A small prod domain ≈ $250-400/mo.

## OpenSearch Serverless vs Domains

- **Serverless** — best for inconsistent traffic, vector workloads, no node mgmt.
- **Domains** — better for steady high traffic, fine-grained control.

## Gotchas

- **OpenSearch ≠ ES.** Same APIs mostly, but newer ES features (8.x) aren't here.
- **Cluster sizing matters.** Use AWS sizing guidance; over-sharding kills perf.
- **Snapshots** to S3 for backup; required for disaster recovery.
- **Security:** put it in a VPC unless you really need public access.

## Related

- [Athena](./athena.md) — sometimes a cheaper alternative for log analysis
- [CloudWatch Logs](../08-monitoring-observability/cloudwatch.md)
- [Bedrock](../09-ml-ai/bedrock.md) — pair vector store with FMs for RAG
