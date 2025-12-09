---
title: Background workers
description: Run long-lived non-HTTP processes — declaring a worker, feeding it work, graceful shutdown, and choosing worker counts.
order: 4
status: stable
audience: developers
category: how-to
product: Nimbus Platform
owners: docs-writers
last_reviewed: 2025-12-09
---

# Background workers

A worker is a service that runs a command forever with no HTTP port: queue consumers, stream processors, schedulers. Workers share the project's [secrets](/reference/cli/secrets), [resources](/reference/config/nimbus-yaml), and private network.

## Declare one

```yaml
services:
  - name: email-consumer
    type: worker
    build: { dockerfile: Dockerfile }
    command: node workers/email.js
    scale: { instances: 2 }
    size: micro
```

If the process exits — crash or clean exit — Nimbus restarts it with exponential backoff. Repeated fast crashes surface in [alerts](/guides/observability/alerts) as restart-loop notifications.

## Feeding a worker

Nimbus has no built-in message queue, so workers typically poll something durable:

- A jobs table in [Nimbus Postgres](/guides/data/nimbus-postgres) claimed with `SELECT ... FOR UPDATE SKIP LOCKED` — simple and transactional; right for modest volumes.
- An external queue service, reached over egress (allowlist the project egress IPs from [Networking](/concepts/networking) if it requires one).
- [Nimbus KV](/guides/data/key-value-store) for lightweight coordination — locks and cursors, not a queue.

Web services can hand work to workers by writing to the same store; for direct calls the other direction, workers can reach web services at `http://<service>.internal:<port>`.

## Graceful shutdown

Deploys and scale-downs send `SIGTERM`, then `SIGKILL` after 30 seconds. Finish the in-flight job and exit:

```js
let draining = false;
process.on("SIGTERM", () => { draining = true; });
// in the poll loop: if (draining) { await finishCurrentJob(); process.exit(0); }
```

Design jobs to be idempotent anyway — a worker killed mid-job means the job runs again.

## Worker vs cron

A worker is for continuous consumption; a [cron service](/concepts/services) is for periodic batches. If the process would sleep most of the time, make it a cron — see the [scheduled jobs tutorial](/tutorials/scheduled-jobs).
