---
title: Scaling
description: Autoscaling web services on CPU or concurrency, fixed scaling for workers, scale-to-zero, and how to load-test a scaling policy.
order: 2
status: stable
audience: developers
category: how-to
product: Nimbus Platform
owners: docs-writers
last_reviewed: 2025-08-12
---

# Scaling

Web services autoscale horizontally between bounds you set; workers scale to a fixed count. All of it lives under the service's `scale` key in [nimbus.yaml](/reference/config/nimbus-yaml).

## Autoscaling web services

```yaml
services:
  - name: api
    type: web
    scale:
      min: 2
      max: 10
      target_cpu: 70        # percent, or use target_concurrency
```

The autoscaler evaluates every 30 seconds. Above target, it adds instances (at most doubling per step); below target for five sustained minutes, it removes one at a time. Scaling events appear in [metrics](/guides/observability/metrics) and can trigger [alerts](/guides/observability/alerts).

## CPU vs concurrency targets

| Target | Scales on | Best for |
| --- | --- | --- |
| `target_cpu` | Average CPU percent across instances | CPU-bound request work |
| `target_concurrency` | Average in-flight requests per instance | I/O-bound services that wait on databases or APIs |

Set exactly one. For `target_concurrency`, measure how many concurrent requests one instance handles at acceptable latency and set the target slightly below it.

## Scale to zero

`min: 0` lets a web service stop entirely when idle. The next request cold-starts an instance, which typically adds one to three seconds. Good for internal tools; avoid it for latency-sensitive public endpoints.

## Workers

Workers use a fixed count — there is no request signal to autoscale on:

```yaml
  - name: consumer
    type: worker
    scale: { instances: 3 }
```

Size worker counts to your queue's throughput; the pattern is discussed in [Background workers](/guides/compute/background-workers).

## Verifying a policy

Load-test against a [preview environment](/concepts/projects-and-environments) and watch instance count, p95 latency, and CPU side by side in [metrics](/guides/observability/metrics). If instances flap up and down, widen the gap between `min` and observed steady-state load, or raise the target.
