---
title: Services
description: The four Nimbus service types — web, worker, cron, and static — what each provides, and how they are declared.
order: 3
status: stable
audience: all
category: concept
product: Nimbus Platform
owners: docs-writers
last_reviewed: 2026-02-10
---

# Services

A service is the deployable unit inside a project. Each entry under `services:` in [nimbus.yaml](/reference/config/nimbus-yaml) declares one, and each is built and scaled independently.

## Service types

| Type | Gets HTTP traffic | Scales | Typical use |
| --- | --- | --- | --- |
| `web` | Yes, behind the load balancer | [Autoscaling](/guides/compute/scaling) between `min` and `max` | APIs, web apps |
| `worker` | No | Fixed instance count | Queue consumers, stream processors |
| `cron` | No | One run per tick | Reports, cleanup jobs |
| `static` | Yes, from the edge | Not applicable | Prebuilt frontend assets |

## Web services

A web service must listen on the port declared under `http.port`, on all interfaces (`0.0.0.0`). Nimbus routes HTTPS traffic to it and polls `http.health_check` to gate deploys — a new version only receives traffic once its health check passes. Getting this wrong is the most common deploy failure; see [Deployment errors](/troubleshooting/deployment-errors).

```yaml
services:
  - name: api
    type: web
    build: { dockerfile: Dockerfile }
    http: { port: 8080, health_check: /healthz }
    scale: { min: 1, max: 4, target_cpu: 70 }
```

## Workers

Workers run the configured `command` continuously and are restarted if they exit. They share the project's secrets and resources, which makes them the natural home for queue consumers — see [Background workers](/guides/compute/background-workers).

## Cron jobs

Cron services run `command` on a `schedule` (standard five-field cron syntax, UTC, minimum interval one minute). Each run gets a fresh instance and its logs are grouped per run. The [scheduled jobs tutorial](/tutorials/scheduled-jobs) builds one end to end.

## Static sites

Static services upload the contents of `publish_dir` produced by an optional `build.command` and serve them from the edge in every region, with immutable caching for fingerprinted assets. See the [static site tutorial](/tutorials/static-site).

## Naming and URLs

Service names are lowercase alphanumerics and hyphens, unique within the project. Web and static services are reachable at `https://<service>-<project>.nimbusapp.example` and through any [custom domains](/guides/compute/custom-domains) you attach.
