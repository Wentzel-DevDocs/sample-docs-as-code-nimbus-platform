---
title: Deploying services
description: How Nimbus builds and rolls out services — Dockerfiles vs buildpacks, build caching and limits, deploying specific services, and zero-downtime rollouts.
order: 1
---

# Deploying services

`nimbus deploy` builds every service in [nimbus.yaml](/reference/config/nimbus-yaml) that changed and rolls each out independently. This page covers what happens in between.

## Builds

Each service declares how it is built:

```yaml
services:
  - name: api
    type: web
    build:
      dockerfile: Dockerfile        # or: buildpack: node
      context: .                    # build context, default "."
```

Builds run on Nimbus build machines with layer caching keyed on the service and branch — unchanged layers (like `npm ci` with an unchanged lockfile) are reused across deploys. Two hard limits apply: builds time out after **30 minutes**, and images must stay under **8 GiB**. Hitting either is covered in [Build failures](/troubleshooting/build-failures).

Because images are keyed on the commit, deploying the same commit to a second environment (the staging → production promotion in [Projects and environments](/concepts/projects-and-environments)) reuses the image instead of rebuilding.

## Deploying a subset

```bash
nimbus deploy                # everything that changed
nimbus deploy api reports    # only these services
nimbus deploy --env staging  # target a different environment
```

## The rollout, step by step

For each web service: new instances start, the platform polls `http.health_check` until it returns `2xx`, traffic shifts to the new instances, then old instances receive `SIGTERM` and get **30 seconds** to drain before `SIGKILL`. Handle `SIGTERM` by finishing in-flight requests and exiting.

Workers and cron services skip the traffic step: new workers start, old ones are signaled the same way.

## Instance sizes

Set per-service CPU and memory:

```yaml
    size: standard-1   # 1 vCPU / 2 GiB. Also: standard-2 (2/4), standard-4 (4/8), micro (0.25/0.5)
```

`micro` suits low-traffic services and crons; scale **out** before scaling **up** for web traffic — see [Scaling](/guides/compute/scaling).

## Deploy history

`nimbus status` shows the live deploy per service; the dashboard lists full history with the builder logs of each `dep_...`. Rollback semantics are described in [Your first deployment](/guides/getting-started/first-deployment).
