---
title: Glossary
description: Short definitions of Nimbus terms — project, environment, service, deploy, region, edge, Store, KV, and Postgres.
order: 7
status: stable
audience: all
category: concept
product: Nimbus Platform
owners: docs-writers
last_reviewed: 2026-07-16
---

# Glossary

Enterprise doc sets often include a glossary so product, support, and engineering share one vocabulary. Definitions here match the rest of this sample site.

| Term | Definition |
| --- | --- |
| **Organization** | Billing and membership boundary. Contains projects. |
| **Project** | One application described by a `nimbus.yaml`, with a primary region and attached resources. |
| **Environment** | Isolated instance of a project (`production`, preview, or custom) with its own config and secrets. |
| **Service** | Deployable unit: `web`, `worker`, `cron`, or `static`. |
| **Deploy** | Immutable build + rollout of one or more services into an environment. |
| **Region** | Geographic capacity zone where service instances run. |
| **Edge** | Globally distributed TLS termination and static asset cache. |
| **Nimbus Store** | S3-compatible object storage attached to a project. |
| **Nimbus KV** | Low-latency key-value store for sessions, flags, and small metadata. |
| **Nimbus Postgres** | Managed PostgreSQL 16 with automated backups. |
| **Buildpack** | Auto-detected build path when no Dockerfile is present. |
| **Health check** | HTTP probe that must pass before traffic shifts to a new deploy. |

See also [Platform overview](/concepts/platform-overview) and [Projects and environments](/concepts/projects-and-environments).
