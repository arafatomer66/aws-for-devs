# ElastiCache

**TL;DR** — Managed Redis or Memcached. In-memory cache or pub/sub. Sub-ms latency. You provision nodes; AWS handles failover, patching, backups.

## What it is

ElastiCache offers two engines:
- **ElastiCache for Redis** — full Redis (data structures, persistence, pub/sub, clustering, geo, streams).
- **ElastiCache for Memcached** — simple K/V cache, multi-threaded, no persistence.

Plus the newer **ElastiCache Serverless** (Redis OSS / Valkey) — fully managed, auto-scaling, no node sizing.

## Why it exists

Some workloads need data in microseconds (sessions, leaderboards, rate-limits, hot product data, queue counters). Managing Redis/Memcached yourself = patching, replication, failover, monitoring. ElastiCache handles all that.

## Key concepts (Redis)

- **Cluster** — one or more shards. Each shard = primary + 0-5 replicas.
- **Node** — one Redis process / EC2 instance.
- **Cluster mode disabled** — single primary + replicas, all data on one node.
- **Cluster mode enabled** — sharded; data split across shards by key hash slot.
- **Replication group** — Redis terminology for the cluster.
- **Backups** — automated snapshots to S3.
- **Multi-AZ** — replicas in different AZs.
- **Encryption in transit / at rest** — TLS, KMS.
- **Auth token** — Redis AUTH password (basic) or RBAC (proper users/permissions).
- **Valkey** — Linux Foundation Redis fork, now an option in ElastiCache (open-source license-friendlier).

## Real-world example

> ShareDeal caches the product detail page:
> - User hits `/product/999`.
> - API checks Redis `GET product:999`. Hit → return.
> - Miss → query Aurora → `SET product:999 EX 600` (10 min TTL) → return.
> - Reduces Aurora reads by ~95%, p99 latency drops from 50ms to 5ms.

Other patterns:
- **Session store** — short TTL, fast access.
- **Leaderboards** — Redis sorted sets (`ZADD`, `ZRANGE`).
- **Rate limiting** — `INCR` with TTL.
- **Pub/sub** — real-time fan-out (chat, notifications).
- **Queue** — lightweight Redis-as-queue (consider SQS for durability).

## Usage

### Create a Redis cluster

```bash
aws elasticache create-replication-group \
  --replication-group-id sharedeal-cache \
  --replication-group-description "Prod cache" \
  --engine redis --engine-version 7.1 \
  --cache-node-type cache.t4g.small \
  --num-cache-clusters 2 \
  --automatic-failover-enabled \
  --multi-az-enabled \
  --transit-encryption-enabled --auth-token 'super-secret' \
  --at-rest-encryption-enabled \
  --cache-subnet-group-name redis-subnets \
  --security-group-ids sg-redis
```

### ElastiCache Serverless (Valkey / Redis OSS)

```bash
aws elasticache create-serverless-cache \
  --serverless-cache-name sharedeal-cache \
  --engine valkey \
  --security-group-ids sg-redis \
  --subnet-ids subnet-a subnet-b subnet-c
```

No node sizing. Bills per GB-hr storage + per ECPU (compute unit).

### Connect (Node, ioredis)

```js
import Redis from "ioredis";
const redis = new Redis({
  host: "sharedeal-cache.xxxx.cache.amazonaws.com",
  port: 6379,
  tls: {},  // TLS in transit
  password: process.env.REDIS_AUTH,
});

await redis.set("product:999", JSON.stringify(product), "EX", 600);
const cached = await redis.get("product:999");
```

## Pricing

- **`cache.t4g.small`** ≈ $14/mo.
- **`cache.m6g.large`** ≈ $87/mo.
- Multi-AZ doubles cost.
- Serverless: $0.125 per GB-hr storage + $0.0034 per million ECPUs.

## Redis vs Memcached

| | Redis | Memcached |
|---|---|---|
| Data types | Strings, hashes, sets, sorted sets, streams, etc. | Strings only |
| Persistence | Yes (snapshots, AOF) | No |
| Replication | Yes | No |
| Pub/sub | Yes | No |
| Multi-threaded | No (cluster shards) | Yes |
| Use | Sessions, leaderboards, queues, rate limits, app cache | Simple page/output cache |

**Default to Redis** unless you have a clear reason for Memcached.

## DAX vs ElastiCache for DynamoDB caching

- **DAX** — write-through cache for DynamoDB specifically. Transparent — no code changes.
- **ElastiCache** — general-purpose; you choose what to cache.

## Gotchas

- **Not durable.** Always have the source of truth elsewhere (DB).
- **Eviction policy matters.** Default `volatile-lru` only evicts keys with TTLs. Set `allkeys-lru` if you cache everything.
- **Connection storms after failover** — use client libs with proper retry/backoff.
- **Cluster mode enabled = different client config.** ioredis/lettuce have specific cluster modes.
- **`KEYS *`** in production = blocks the cluster. Use `SCAN`.
- **TLS in transit adds ~ms latency** but it's table-stakes.
- **Backup snapshots count against S3** for storage cost.

## Related

- [MemoryDB](./memorydb.md) — durable Redis-API alternative
- [DAX](#) — DynamoDB-specific cache
- [SQS](../06-messaging-integration/sqs.md) — for actual queues
