---
title: Managed Postgres
description: Provision a Postgres instance, attach it to services, connect with psql, and understand backups and point-in-time recovery.
nav_order: 2
---
Nimbus runs Postgres for you: provisioning, storage, backups, minor-version upgrades. You bring the schema.

## Plans

| Plan | vCPU | Memory | Storage | Fit |
| --- | --- | --- | --- | --- |
| `dev` | shared | 512 MB | 5 GB | development, previews |
| `standard-2` | 2 | 4 GB | 50 GB | most production apps |
| `standard-4` | 4 | 8 GB | 100 GB | heavier workloads |
| `business-8` | 8 | 32 GB | 500 GB | high traffic, 30-day PITR |

## Create and attach

```bash
nimbus pg create maindb --plan standard-2
nimbus pg attach maindb --service web
```

`attach` injects a `DATABASE_URL` [environment variable](/reference/config/environment-variables) whose host is a [private-network](/guides/networking/private-networks) address - the database is never exposed publicly. Each [environment](/guides/getting-started/environments) gets its own instance; staging can never read production.

## Connect and inspect

```bash
nimbus pg psql maindb            # interactive psql session
nimbus pg url maindb             # print the connection string (for one-off tools)
nimbus pg status maindb          # plan, storage used, connection count
```

Connections are TLS-required. Connection limits scale with the plan; if you exhaust them, add a pooler in your app before upsizing - symptoms and fixes in [performance](/troubleshooting/performance).

## Backups and recovery

Every instance gets continuous backups with **point-in-time recovery: 7 days** on `dev` and `standard-*`, **30 days** on `business-8`.

```bash
nimbus pg restore maindb --at "2026-07-15T09:30:00Z" --new-name maindb-recovered
```

Restores always create a *new* instance so you can inspect before switching - re-point services by attaching the recovered instance. Migrations fit naturally in a [one-off job](/guides/compute/jobs): `nimbus jobs run -- node scripts/migrate.js`.

## Maintenance

Minor versions are applied automatically in a low-traffic window with a connection blip of a few seconds; major-version upgrades are user-initiated via `nimbus pg upgrade`. Watch `db.maintenance.scheduled` [events](/reference/api/events) to know ahead of time.
