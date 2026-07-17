---
title: Key-value store
description: A low-latency KV namespace per environment - commands, TTLs, size limits, and the consistency model.
nav_order: 3
---
The key-value store is for small, hot data: sessions, feature flags, rate counters, caches. It is not a document database - values are opaque bytes and the only index is the key.

## Create and attach

```bash
nimbus kv create sessions
nimbus kv attach sessions --service web
```

`attach` injects `NIMBUS_KV_URL` (a [private-network](/guides/networking/private-networks) address) and `NIMBUS_KV_NAMESPACE`. Client libraries exist for Node, Python, and Go; the wire protocol is documented for everything else.

## Basic operations

```bash
nimbus kv set sessions user:42 '{"name":"Ada"}' --ttl 3600
nimbus kv get sessions user:42
nimbus kv delete sessions user:42
nimbus kv list sessions --prefix user:
```

The same verbs exist in the SDKs (`kv.set(key, value, { ttl })`). `list` is prefix-only and paginated - if you need secondary indexes or queries, that is [Postgres](/guides/data/managed-postgres)'s job.

## Limits

| Limit | Value |
| --- | --- |
| Key size | 512 bytes |
| Value size | 1 MB |
| Namespace size | 10 GB |
| TTL | optional, per key, seconds granularity |

Expired keys stop being readable immediately and are garbage-collected in the background (they may briefly count toward namespace size).

## Consistency

Reads and writes are **strongly consistent within a region**: a `get` after a completed `set` in the same [region](/concepts/regions-and-zones) sees the new value. There is no cross-region replication - a namespace lives in its project's region, and workloads in other regions pay cross-region latency to reach it.

## When to use what

- Ephemeral, small, hot, key-addressed -> **KV**.
- Relational, queryable, durable source of truth -> [Postgres](/guides/data/managed-postgres).
- Large blobs -> [object storage](/guides/data/object-storage) (a KV value maxes out at 1 MB).

Counters are a natural fit: `nimbus kv incr sessions rate:api:42 --ttl 60` increments atomically, which is exactly the primitive a per-user rate limiter needs.
