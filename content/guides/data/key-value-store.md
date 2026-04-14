---
title: Key-value store (Nimbus KV)
description: Attach a KV namespace for caching, sessions, and counters — the API, TTLs, atomic increments, size limits, and consistency model.
order: 2
status: stable
audience: developers
category: how-to
product: Nimbus Platform
owners: docs-writers
last_reviewed: 2026-04-14
---

# Key-value store (Nimbus KV)

Nimbus KV is a low-latency key-value store for the small, hot data that does not belong in Postgres: sessions, caches, rate counters, feature flags. Namespaces are replicated to every region, so reads are fast everywhere.

## Attach a namespace

```yaml
resources:
  - name: cache
    type: kv
```

Services receive `NIMBUS_KV_CACHE_URL` and `NIMBUS_KV_CACHE_KEY`.

## Basic operations

```bash
# set with a 15-minute TTL
curl -X PUT "$NIMBUS_KV_CACHE_URL/keys/session:u31?ttl=900" \
  -H "Authorization: Bearer $NIMBUS_KV_CACHE_KEY" \
  -d '{ "user": 31, "role": "admin" }'

# get
curl "$NIMBUS_KV_CACHE_URL/keys/session:u31" \
  -H "Authorization: Bearer $NIMBUS_KV_CACHE_KEY"

# delete
curl -X DELETE "$NIMBUS_KV_CACHE_URL/keys/session:u31" \
  -H "Authorization: Bearer $NIMBUS_KV_CACHE_KEY"
```

Values are opaque bytes (JSON by convention). A missing key returns `404`. Keys can be listed by prefix: `GET /keys?prefix=session:`.

## TTLs

Pass `ttl` (seconds) on any write; the key vanishes when it expires. Omitted means the key lives until deleted. TTLs are the idiomatic way to build caches — do not build your own sweeper.

## Atomic counters

```bash
curl -X POST "$NIMBUS_KV_CACHE_URL/keys/rl:ip:203.0.113.9/incr?by=1&ttl=60" \
  -H "Authorization: Bearer $NIMBUS_KV_CACHE_KEY"
# { "value": 7 }
```

`incr` is atomic across regions, which makes sliding-window rate limiting a three-line function.

## Limits and consistency

| Limit | Value |
| --- | --- |
| Key size | 512 bytes |
| Value size | 400 KB |
| Writes per key | 1 per second sustained |

Reads in the writing region are read-your-writes; other regions converge within about a second. KV is **not** for data you cannot lose or re-derive — that is [Nimbus Postgres](/guides/data/nimbus-postgres). Values above the size limit belong in [Store](/guides/data/object-storage).
