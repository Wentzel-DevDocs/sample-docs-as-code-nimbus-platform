---
title: FAQ
description: Frequently asked questions — pricing model, sleeping free services, monorepos, Docker Compose, data export, region moves, and support.
order: 4
status: stable
audience: all
category: troubleshooting
product: Nimbus Platform
owners: docs-writers
last_reviewed: 2026-04-14
---

# FAQ

## Does the free tier sleep?

Free-plan web services scale to zero after 15 idle minutes and cold-start on the next request (one to three seconds), and the `dev-0` Postgres plan pauses after 7 idle days. Paid plans keep `min` instances warm — set `scale.min: 1` and any paid plan to never sleep. See [Scaling](/guides/compute/scaling).

## How am I billed?

Per instance-second by [size](/guides/compute/deploying-services), plus resource plans ([Postgres](/guides/data/nimbus-postgres)) and storage/egress for [Store](/guides/data/object-storage). The dashboard's **Usage** tab shows a live month-to-date figure per project. Cron runs bill only while running.

## Can I deploy a monorepo with several services?

Yes — one `nimbus.yaml`, several services, each with its own `build.context`:

```yaml
services:
  - { name: api,  type: web,    build: { context: apps/api,  dockerfile: Dockerfile }, http: { port: 8080, health_check: /healthz } }
  - { name: site, type: static, build: { context: apps/site, buildpack: node, command: npm run build }, publish_dir: apps/site/dist }
```

`nimbus deploy` rebuilds only services whose context changed.

## Can I run docker-compose.yml?

Not directly — Nimbus deploys services, not compose stacks. Each compose service becomes an entry under `services:`; compose `depends_on` links become calls over [internal DNS](/concepts/networking); databases move to [managed Postgres](/guides/data/nimbus-postgres) rather than a container.

## How do I get my data out?

Postgres: `nimbus db connect` and `pg_dump`, or a [restore](/guides/data/nimbus-postgres) into a copy you dump at leisure. Store: list and `GET` objects via the [storage API](/reference/api/storage), or `nimbus store sync` in reverse. KV holds derivable data by design — see [its guide](/guides/data/key-value-store).

## Can I change my project's region later?

Yes, with caveats: compute moves on the next deploy, data does not move automatically, and your egress IPs change. The procedure is in [Regions and the edge](/concepts/regions-and-edge); the allowlist gotcha is in [Networking issues](/troubleshooting/networking).

## Where do I get help?

Check the relevant troubleshooting page first ([builds](/troubleshooting/build-failures), [deploys](/troubleshooting/deployment-errors), [networking](/troubleshooting/networking)), then email `support@nimbus.example` with the project name and any `dep_...` or `req_...` IDs — they let support find the exact build or request.
