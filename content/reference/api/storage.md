---
title: Storage API
description: REST endpoints for Nimbus Store objects and signed URLs, and for Nimbus KV keys, TTLs, and atomic increments.
order: 4
status: stable
audience: developers
category: reference
product: Nimbus Platform
owners: docs-writers
last_reviewed: 2026-01-13
---

# Storage API

[Store buckets](/guides/data/object-storage) and [KV namespaces](/guides/data/key-value-store) each expose a small HTTP API at the base URL injected into your services. Both authenticate with the injected bucket- or namespace-scoped key, **not** a personal access token.

## Store

Base: `NIMBUS_STORE_<NAME>_URL`, auth: `Bearer NIMBUS_STORE_<NAME>_KEY`.

| Method and path | Purpose |
| --- | --- |
| `PUT /objects/{key}` | Write an object (body = bytes); up to 5 GiB per request |
| `GET /objects/{key}` | Read an object |
| `HEAD /objects/{key}` | Metadata only: size, `ETag`, `Last-Modified` |
| `DELETE /objects/{key}` | Delete |
| `GET /objects?prefix=&limit=&starting_after=` | List keys, paginated like the [core API](/reference/api/overview) |
| `POST /sign` | Mint a signed URL |
| `POST /multipart/{key}` | Start a multipart upload for objects over 5 GiB |

`POST /sign` body: `{ "method": "GET"|"PUT", "key": "...", "expires_in": 60–86400 }` → `{ "url": "..." }`. Signed URLs need no auth header and are safe to hand to browsers.

Conditional reads are supported: send `If-None-Match: <etag>` and get `304` when unchanged.

## KV

Base: `NIMBUS_KV_<NAME>_URL`, auth: `Bearer NIMBUS_KV_<NAME>_KEY`.

| Method and path | Purpose |
| --- | --- |
| `PUT /keys/{key}?ttl=<s>` | Set a value; optional TTL in seconds |
| `GET /keys/{key}` | Read (404 when missing or expired) |
| `DELETE /keys/{key}` | Delete |
| `GET /keys?prefix=&limit=&starting_after=` | List key names |
| `POST /keys/{key}/incr?by=<n>&ttl=<s>` | Atomic increment; creates at `by` when missing → `{ "value": n }` |

Limits (512-byte keys, 400 KB values, one sustained write per key per second) and the consistency model are described in the [KV guide](/guides/data/key-value-store).

## Errors

Both APIs use the same envelope and codes as the [core API](/reference/api/overview), plus `413 too_large` for oversized values and `412 precondition_failed` for conditional requests.
