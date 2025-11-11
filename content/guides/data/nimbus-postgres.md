---
title: Managed Postgres
description: Provision a Nimbus Postgres database, connect from services and locally, understand plans, backups, point-in-time recovery, and upgrades.
order: 3
status: stable
audience: developers
category: how-to
product: Nimbus Platform
owners: docs-writers
last_reviewed: 2025-11-11
---

# Managed Postgres

Nimbus Postgres is managed PostgreSQL 16: provisioning, backups, monitoring, and failover handled by the platform, plain Postgres for everything else — extensions like `pgcrypto` and `postgis` included.

## Attach a database

```yaml
resources:
  - name: maindb
    type: postgres
    plan: standard-1
```

Deploying provisions the database and injects `NIMBUS_DATABASE_URL` (a full `postgres://` connection string) into every service in the project. With `per_environment: true`, [previews](/concepts/projects-and-environments) each get their own empty database instead of sharing production's.

## Plans

| Plan | vCPU / RAM | Storage | Notes |
| --- | --- | --- | --- |
| `dev-0` | shared / 1 GiB | 5 GiB | Free tier; pauses after 7 idle days |
| `standard-1` | 1 / 4 GiB | 50 GiB | Daily backups + PITR |
| `standard-2` | 2 / 8 GiB | 100 GiB | Daily backups + PITR |
| `standard-4` | 4 / 16 GiB | 250 GiB | Daily backups + PITR, HA standby |

Changing `plan` and deploying resizes in place with seconds of failover-level interruption on standard plans.

## Connecting

From services, just use the injected URL. Connections are pooled platform-side; keep your app-side pool modest (5–10 per instance) so [autoscaling](/guides/compute/scaling) cannot exhaust Postgres connection slots.

Locally, tunnel through the CLI rather than exposing the database:

```bash
nimbus db connect maindb
# psql session opens
nimbus db url maindb --print   # prints a localhost-tunneled URL for GUI tools
```

## Backups and recovery

Standard plans take a daily snapshot (kept 14 days) and stream WAL for **point-in-time recovery** to any moment in the last 7 days:

```bash
nimbus db restore maindb --at "2026-07-15T09:30:00Z" --into maindb-restored
```

Restores always create a **new** database resource, so recovery never overwrites what you are diagnosing. Wire the restored copy in by pointing `NIMBUS_DATABASE_URL` at it via a [secret override](/reference/cli/secrets) or by renaming resources in `nimbus.yaml`.

## Migrations

Run schema migrations from a release step, not at process boot — a [cron service](/concepts/services) triggered manually or your CI calling `nimbus run api -- npm run migrate` keeps migrations single-flight while instances scale. The [Postgres tutorial](/tutorials/postgres-app) shows the full pattern.
