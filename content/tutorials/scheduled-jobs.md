---
title: Run scheduled jobs
description: Build a cron service — schedule syntax, writing a well-behaved job, reading per-run logs, overlap rules, and failure alerts.
order: 3
---

# Run scheduled jobs

Cron services run a command on a schedule. This tutorial adds a nightly report job to an existing project and covers the operational details that make crons boring in the good way.

## 1. Declare the cron

```yaml
services:
  - name: nightly-report
    type: cron
    build: { dockerfile: Dockerfile }
    schedule: "0 6 * * *"        # 06:00 UTC daily
    command: node jobs/report.js
    size: micro
```

Schedules are five-field cron expressions evaluated in **UTC**, minimum interval one minute. Common patterns:

| Expression | Runs |
| --- | --- |
| `*/5 * * * *` | Every five minutes |
| `0 * * * *` | Hourly, on the hour |
| `0 6 * * 1` | Mondays 06:00 UTC |
| `30 2 1 * *` | Monthly, on the 1st at 02:30 UTC |

## 2. Write a well-behaved job

```js
// jobs/report.js
const started = Date.now();
try {
  const rows = await buildReport();          // your work
  console.log(JSON.stringify({ level: "info", msg: "report done", rows, ms: Date.now() - started }));
  process.exit(0);
} catch (err) {
  console.error(JSON.stringify({ level: "error", msg: "report failed", err: String(err) }));
  process.exit(1);
}
```

Exit `0` on success and non-zero on failure — the exit code is what marks the run failed and drives [alerts](/guides/observability/alerts). Make the job idempotent: a run that dies mid-way will simply run again at the next tick.

## 3. Rules the platform enforces

- **No overlap:** if a run is still going when the next tick arrives, the tick is skipped and logged as `skipped: previous run in progress`.
- Each run gets a fresh instance; nothing persists between runs except what you write to [Postgres](/guides/data/nimbus-postgres), [Store](/guides/data/object-storage), or [KV](/guides/data/key-value-store).
- A run that exceeds its schedule interval fires the overrun alert.

## 4. Observe runs

```bash
nimbus logs nightly-report --run latest
nimbus deploy nightly-report && nimbus run nightly-report -- node jobs/report.js   # trigger once, manually
```

The dashboard lists every run with duration and exit status. The failure alert (on by default) emails on any non-zero exit — wire it to a webhook per [Alerts](/guides/observability/alerts) if you page on it.
