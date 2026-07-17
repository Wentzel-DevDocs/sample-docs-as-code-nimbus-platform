---
title: Volumes API
description: Persistent disks for services - create, attach, snapshot, and restore, with the single-attach rule explained.
nav_order: 3
---
A volume is a persistent disk that survives deploys and restarts, for the rare service that needs local durable state (an embedded database, a search index). Most state belongs in [Postgres](/guides/data/managed-postgres) or [object storage](/guides/data/object-storage) instead.

## The volume object

```json
{
  "id": "vol_3Kd9s",
  "name": "search-index",
  "size_gb": 50,
  "attached_to": "svc_7Hq2m",
  "mount_path": "/data",
  "zone": "us-east-a",
  "created_at": "2026-06-11T08:20:00Z"
}
```

## Endpoints

| Method and path | Purpose |
| --- | --- |
| `GET /v1/volumes` | List volumes ([paginated](/reference/api/overview)) |
| `POST /v1/volumes` | Create - `name`, `size_gb` (1-500), optional `zone` |
| `PATCH /v1/volumes/:id` | Grow `size_gb` (shrinking is not supported) |
| `POST /v1/volumes/:id/attach` | Attach - `service`, `mount_path` |
| `POST /v1/volumes/:id/detach` | Detach (service must be stopped or restarting) |
| `DELETE /v1/volumes/:id` | Delete - must be detached; irreversible |
| `POST /v1/volumes/:id/snapshots` | Create a snapshot |
| `GET /v1/volumes/:id/snapshots` | List snapshots |
| `POST /v1/snapshots/:id/restore` | Restore into a **new** volume |

## The single-attach rule

A volume attaches to **one service** and pins it to the volume's [zone](/concepts/regions-and-zones); a service with a volume runs a single instance in that zone. That trades away zone-spread and [autoscaling](/guides/compute/scaling) - which is why the managed data stores, which have their own replication, are almost always the better answer.

## Snapshots

Snapshots are consistent point-in-time copies, billed by stored size. Restores never overwrite in place:

```bash
curl -X POST "https://api.nimbus.example/v1/snapshots/snap_1Fh7q/restore?project=storefront&env=production" \
  -H "Authorization: Bearer nbt_..." \
  -d '{ "name": "search-index-restored" }'
```

Attach the restored volume to swap it in. Snapshot before risky migrations; a `volume.snapshot.created` [event](/reference/api/events) confirms completion.
