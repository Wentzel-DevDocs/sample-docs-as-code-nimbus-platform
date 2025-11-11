---
title: API overview
description: The Nimbus REST API — base URL, request conventions, pagination, error envelope, and rate limits.
order: 1
status: stable
audience: developers
category: reference
product: Nimbus Platform
owners: docs-writers
last_reviewed: 2025-11-11
---

# API overview

Everything the dashboard and CLI do goes through this REST API, and you can call it directly for automation the CLI does not cover: custom dashboards, deployment bots, audit tooling.

## Base URL

```
https://api.nimbus.example/v1
```

Requests and responses are JSON. Authentication is a bearer [personal access token](/reference/api/authentication) on every request.

## Conventions

- Resources are nouns with IDs: projects `prj_...`, services `svc_...`, deployments `dep_...`, resources `res_...`.
- `GET` reads, `POST` creates or acts, `PATCH` updates, `DELETE` removes.
- Timestamps are Unix seconds; durations are seconds unless a field name says otherwise.
- Unknown response fields must be ignored — additive changes ship without a version bump.

## Pagination

List endpoints take `limit` (1–100, default 20) and `starting_after=<id>`, returning:

```json
{ "data": [ ... ], "has_more": true }
```

Pass the last item's `id` as `starting_after` for the next page; stop at `has_more: false`.

## Errors

```json
{ "error": { "code": "not_found", "message": "no such service", "request_id": "req_77ab21" } }
```

| Status | Code | Meaning |
| --- | --- | --- |
| 400 | `invalid_request` | Malformed body or parameters |
| 401 | `unauthenticated` | Missing, expired, or revoked token |
| 403 | `forbidden` | Token scope or role insufficient |
| 404 | `not_found` | No such resource |
| 409 | `conflict` | State conflict, such as deploying during an in-flight deploy |
| 429 | `rate_limited` | Over the rate limit; honor `Retry-After` |
| 500 | `internal` | Nimbus-side failure; retry with backoff |

Include `request_id` when contacting support.

## Rate limits

600 requests per minute per token, with `X-RateLimit-Remaining` and `X-RateLimit-Reset` headers on every response. Metrics queries (see [Metrics](/guides/observability/metrics)) count double.

## Endpoint groups

[Authentication](/reference/api/authentication) · [Projects, services & deployments](/reference/api/services) · [Storage](/reference/api/storage)
