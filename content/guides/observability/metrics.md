---
title: Metrics
description: The metrics Nimbus records per service — requests, latency percentiles, CPU, memory, instance count — and how to query them from the dashboard or API.
order: 2
status: stable
audience: developers
category: how-to
product: Nimbus Platform
owners: docs-writers
last_reviewed: 2026-03-10
---

# Metrics

Every service gets a standard metrics set with no instrumentation required, recorded at 15-second resolution and kept for 30 days.

## What is recorded

| Metric | Applies to | Notes |
| --- | --- | --- |
| `requests` | web | Count, broken down by status class (2xx/3xx/4xx/5xx) |
| `latency_p50`, `latency_p95`, `latency_p99` | web | Measured at the router, so network time is included |
| `cpu` | all | Percent of the instance [size](/guides/compute/deploying-services) |
| `memory` | all | Working set; approaching the limit precedes OOM kills |
| `instances` | all | Running count — the [autoscaler's](/guides/compute/scaling) visible output |
| `runs`, `run_duration` | cron | Per scheduled run |

## Reading them

The dashboard's **Metrics** tab charts everything with overlays for deploys and scaling events — the fastest way to answer "did that deploy change latency". For your own dashboards, the same series are queryable over the [REST API](/reference/api/overview):

```bash
curl "https://api.nimbus.example/v1/services/svc_a1b2/metrics?metric=latency_p95&window=1h&step=60" \
  -H "Authorization: Bearer nbp_2f8a1example"
```

```json
{ "metric": "latency_p95", "unit": "ms", "points": [[1768646400, 84], [1768646460, 91]] }
```

## Reading them well

- Watch **p95/p99**, not averages; averages hide the pain.
- Rising `memory` with flat traffic is the classic leak signature — restart-and-climb sawtooths mean OOM kills, which also appear in [logs](/guides/observability/logs).
- `instances` pinned at `max` with high `cpu` means the ceiling is too low; see [Scaling](/guides/compute/scaling).
- 4xx spikes are usually your clients; 5xx spikes are usually you.

To be told instead of watching, set thresholds on any of these in [Alerts](/guides/observability/alerts).
