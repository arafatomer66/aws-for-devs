# MemoryDB for Redis / Valkey

**TL;DR** — Like ElastiCache (Redis API) but with **multi-AZ durability and strong consistency**. Use it as a primary database, not just a cache.

## What it is

Redis-compatible (Valkey now) in-memory database that durably persists writes across multiple AZs via a transactional log. Sub-ms read latency, single-digit-ms write latency, **no data loss on failover**.

## MemoryDB vs ElastiCache for Redis

| | ElastiCache Redis | MemoryDB |
|---|---|---|
| Latency | Sub-ms | Sub-ms reads, ms writes |
| Durability | Best-effort (snapshots/AOF) | Full multi-AZ durable |
| Use as primary DB | No, lossy on failover | **Yes** |
| Cost | Lower | Higher |

If you want Redis-as-a-cache → ElastiCache.
If you want Redis-as-a-database → MemoryDB.

## Real-world example

> A real-time leaderboard service:
> - Writes (scores) must not be lost.
> - Reads must be sub-ms (live ranking pages).
> - MemoryDB stores sorted sets, durably across 3 AZs.
> - On node failure, transactional log replays on standby — no data loss.

## Usage

Same Redis clients, same data structures. Connection looks identical to ElastiCache.

```bash
aws memorydb create-cluster \
  --cluster-name leaderboard \
  --node-type db.r6g.large --num-shards 1 --num-replicas-per-shard 2 \
  --acl-name default \
  --subnet-group-name memorydb-subnets \
  --security-group-ids sg-0123 \
  --tls-enabled
```

## Pricing

- ~$0.30/hr per `db.r6g.large` (more than ElastiCache for same instance).
- Plus data write to transaction log charges.

## Gotchas

- **Pricier than ElastiCache.** Don't pay for durability you don't need.
- **TLS in transit required.** Plan for it.
- **One writer per shard** — but multi-shard clusters scale writes.

## Related

- [ElastiCache](./elasticache.md)
- [DynamoDB](./dynamodb.md)
