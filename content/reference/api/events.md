---
title: Events API
description: The activity feed - event object, types, filtering, and how to poll it as an audit trail or automation trigger.
nav_order: 4
---
Every meaningful change in a project emits an event: deploys, scaling, domain verification, job outcomes, token creation. The feed is the [audit trail](/concepts/identity-and-access) and a polling-friendly automation hook.

## The event object

```json
{
  "id": "evt_9s2Lq",
  "type": "release.created",
  "actor": { "kind": "user", "id": "usr_4Km1x", "name": "ada@example.com" },
  "project": "storefront",
  "env": "production",
  "resource": { "kind": "release", "id": "r-42", "service": "web" },
  "data": { "message": "ship checkout v2" },
  "created_at": "2026-07-16T14:03:22Z"
}
```

`actor.kind` is `user`, `token` (a [service token](/concepts/identity-and-access) acted), or `system` (Nimbus itself, e.g. autoscaling).

## Common event types

| Type | Fired when |
| --- | --- |
| `release.created` / `release.failed` | A [deploy](/reference/cli/nimbus-deploy) lands or fails |
| `service.scaled` / `service.rollback` | [Scaling](/guides/compute/scaling) changed, release rolled back |
| `job.completed` / `job.failed` | A [job](/guides/compute/jobs) run finished |
| `domain.verified` / `domain.failed` | [Custom domain](/guides/networking/domains) verification resolved |
| `db.maintenance.scheduled` | [Postgres](/guides/data/managed-postgres) maintenance planned |
| `volume.snapshot.created` | A [snapshot](/reference/api/volumes) completed |
| `token.created` / `token.revoked` | Service tokens changed |

## Listing and filtering

```bash
curl "https://api.nimbus.example/v1/events?project=storefront&env=production&type=release.failed&since=2026-07-01T00:00:00Z" \
  -H "Authorization: Bearer nbt_..."
```

Filters: `type` (repeatable), `service`, `actor`, `since`/`until`. Results are newest-first and [paginated](/reference/api/overview).

## Retention and polling

Events are retained for **90 days**. There are no push webhooks in the current API version - poll with `since` set to your last seen `created_at` (events are strictly ordered per project, so this never misses). Stay well inside the [rate limit](/reference/api/overview): once a minute is plenty for alerting on `release.failed` or `job.failed`.
