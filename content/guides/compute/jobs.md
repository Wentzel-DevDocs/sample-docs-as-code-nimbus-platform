---
title: Jobs
description: Run one-off commands and cron-scheduled tasks with the same image and config as your service.
nav_order: 2
---
Jobs run a command to completion instead of serving traffic: migrations, imports, nightly reports. They use the same built image and [environment variables](/reference/config/environment-variables) as the service in the same `nimbus.yaml`.

## One-off jobs

```bash
nimbus jobs run -- node scripts/migrate.js
```

The command runs on a fresh instance, streams its output to your terminal, and exits with the command's exit code - handy in CI gates. Add `--detach` to return immediately and follow later with [`nimbus logs`](/reference/cli/nimbus-logs).

## Scheduled jobs

Declare them under `jobs:` in `nimbus.yaml` with a five-field cron expression, evaluated in **UTC**:

```yaml
jobs:
  - name: nightly-report
    schedule: "0 3 * * *"     # 03:00 UTC daily
    command: ["node", "scripts/report.js"]
    size: small
  - name: cache-warm
    schedule: "*/15 * * * *"  # every 15 minutes
    command: ["node", "scripts/warm.js"]
```

Schedules take effect on the next deploy. List and inspect them:

```bash
nimbus jobs list
nimbus jobs history nightly-report
```

## Timeouts, retries, and overlap

- Default timeout is **15 minutes**; raise it per job with `timeout: 2h`, up to **24 hours**.
- A job that exits non-zero is retried twice with exponential backoff, then marked `failed` (and emits a `job.failed` [event](/reference/api/events)).
- If a run is still going when the next tick arrives, the new run is **skipped**, not queued - design scheduled work to be idempotent.

## Jobs are not services

A job has no port, no health check, and no URL. If your "job" needs to receive requests, it is a [service](/guides/compute/services); if a service needs periodic work, pair it with a scheduled job rather than an in-process timer, so the work survives restarts and scale-to-zero. A complete example is in the [scheduled jobs tutorial](/tutorials/scheduled-jobs).
