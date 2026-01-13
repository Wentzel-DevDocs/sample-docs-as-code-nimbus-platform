---
title: nimbus.yaml
description: The complete nimbus.yaml reference — project keys, every service field by type, scale and size options, and resource declarations.
order: 1
status: stable
audience: developers
category: reference
product: Nimbus Platform
owners: docs-writers
last_reviewed: 2026-01-13
---

# nimbus.yaml

`nimbus.yaml` at the repository root declares everything about a project. Deploys read it from the uploaded source, so the file in your working tree is always the truth.

## Project keys

| Key | Type | Required | Description |
| --- | --- | --- | --- |
| `name` | string | yes | Project name: lowercase, digits, hyphens |
| `region` | string | yes | One of the [regions](/concepts/regions-and-edge): `iad`, `sfo`, `fra`, `sin`, `syd` |
| `services` | list | yes | At least one service |
| `resources` | list | no | Attached storage and databases |

## Service fields (all types)

| Field | Type | Description |
| --- | --- | --- |
| `name` | string | Unique in the project |
| `type` | string | `web`, `worker`, `cron`, or `static` — see [Services](/concepts/services) |
| `build.dockerfile` | string | Path to a Dockerfile (mutually exclusive with `buildpack`) |
| `build.buildpack` | string | `node`, `python`, `go`, or `ruby` |
| `build.context` | string | Build context directory, default `.` |
| `command` | string | Process to run (workers and crons; optional override for web) |
| `size` | string | `micro`, `standard-1`, `standard-2`, `standard-4` |
| `env` | map | Plain environment variables — see [Environment variables](/reference/config/environment-variables) |

## Web-only fields

| Field | Description |
| --- | --- |
| `http.port` | Port the process listens on (required) |
| `http.health_check` | Path polled during rollouts (required) |
| `scale.min`, `scale.max` | Autoscaling bounds; `min: 0` enables scale-to-zero |
| `scale.target_cpu` or `scale.target_concurrency` | Exactly one — see [Scaling](/guides/compute/scaling) |
| `domains` | List of [custom domains](/guides/compute/custom-domains) |

## Worker, cron, and static fields

| Field | Applies to | Description |
| --- | --- | --- |
| `scale.instances` | worker | Fixed instance count |
| `schedule` | cron | Five-field cron expression, UTC, minimum interval one minute |
| `build.command` | static | Command that produces the site |
| `publish_dir` | static | Directory to upload and serve |

## Resources

| Field | Description |
| --- | --- |
| `name` | Unique in the project; determines injected variable names |
| `type` | `store`, `kv`, or `postgres` |
| `plan` | Postgres only: `dev-0`, `standard-1`, `standard-2`, `standard-4` |
| `public_read` | Store only: anonymous, edge-cached `GET`s |
| `per_environment` | Provision a fresh copy per [environment](/concepts/projects-and-environments) |

## A complete example

```yaml
name: my-app
region: iad
services:
  - name: api
    type: web
    build: { dockerfile: Dockerfile }
    http: { port: 8080, health_check: /healthz }
    scale: { min: 2, max: 10, target_cpu: 70 }
    size: standard-1
    env: { LOG_LEVEL: info }
    domains: [api.example.com]
  - name: email-consumer
    type: worker
    build: { dockerfile: Dockerfile }
    command: node workers/email.js
    scale: { instances: 2 }
  - name: nightly-report
    type: cron
    build: { dockerfile: Dockerfile }
    schedule: "0 6 * * *"
    command: node jobs/report.js
resources:
  - { name: maindb, type: postgres, plan: standard-1 }
  - { name: uploads, type: store }
  - { name: cache, type: kv }
```

Validate without deploying: `nimbus deploy --dry-run` (see the [deploy reference](/reference/cli/deploy)).
