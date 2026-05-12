---
title: Deployment errors
description: Diagnosing rollouts that build but fail to go live — health check timeouts, port binding mistakes, crash loops, OOM kills, and stuck deploys.
order: 2
status: stable
audience: all
category: troubleshooting
product: Nimbus Platform
owners: docs-writers
last_reviewed: 2026-05-12
---

# Deployment errors

These failures happen after a successful build: new instances start but never go `live`. The previous version keeps serving throughout — see the rollout mechanics in [Deploying services](/guides/compute/deploying-services). Diagnosis starts with the failing instances' logs:

```bash
nimbus logs <service> --deploy <dep_id>
```

## "waiting for health check" then failure

The platform polled `http.health_check` and never got a `2xx`. In rough order of likelihood:

1. **Wrong port.** The process must listen on exactly `http.port` from [nimbus.yaml](/reference/config/nimbus-yaml). If the app reads `PORT` from the environment, make the two agree.
2. **Bound to localhost.** Listening on `127.0.0.1` is unreachable from the router — bind `0.0.0.0`.
3. **No such route.** The health path 404s. Add the endpoint; it should be dependency-free and instant (see [Set up a project](/guides/getting-started/project-setup)).
4. **Slow start.** The app takes longer to boot than the check window. Defer heavy warm-up until after the listener is up, and never run migrations at boot — see the [Postgres tutorial](/tutorials/postgres-app).

## Crash loop

Log lines show the process starting and exiting repeatedly. The exit reason is in the log just before each restart — typically a missing [secret](/reference/cli/secrets) (`undefined is not a connection string`), a bad `command`, or an unhandled exception during startup. Workers loop the same way; the restart-loop [alert](/guides/observability/alerts) fires after three crashes in five minutes.

## OOM killed

Lifecycle lines read `killed: out of memory`. The instance exceeded its [size](/guides/compute/deploying-services) memory. Either raise `size` or fix the leak — a sawtooth `memory` chart in [metrics](/guides/observability/metrics) distinguishes steady growth (leak) from a spike (undersized).

## Deploy seems stuck in `queued`

One build runs per service at a time; a queued deploy is waiting for the in-flight one. `nimbus status` shows what is running; a superseding push cancels the older queued deploy automatically.

## After the rollout: 502/503 from the URL

`502` means no healthy instances (mid-crash-loop, or scaled to zero and cold-starting — see [Scaling](/guides/compute/scaling)); `503` with a `Retry-After` means the platform is load-shedding a saturated service. Both are visible immediately in `requests` by status class in [metrics](/guides/observability/metrics). Routing and DNS problems are on the next page: [Networking](/troubleshooting/networking).
