---
title: Alerts
description: Create threshold alerts on metrics, get platform event alerts for crashes and failed deploys, and route notifications to email or webhooks.
order: 3
status: stable
audience: developers
category: how-to
product: Nimbus Platform
owners: docs-writers
last_reviewed: 2025-11-11
---

# Alerts

Alerts watch [metrics](/guides/observability/metrics) and platform events so you do not have to. They are configured per project under **Project → Observability → Alerts**.

## Threshold alerts

A threshold alert names a service, a metric, a comparison, and a sustain window:

| Field | Example |
| --- | --- |
| Service / metric | `api` / `latency_p95` |
| Condition | `> 500` (ms) |
| Sustained for | `5m` |
| Notify | email, webhook |

The sustain window is what keeps alerts quiet during blips — a single slow scrape never pages anyone. Alerts fire once on breach and once on recovery.

Recommended starter set for a web service: `latency_p95 > 2× normal for 5m`, `requests(5xx) > 1% of total for 5m`, `memory > 90% for 10m`, and `instances == max for 15m` (your [scaling](/guides/compute/scaling) ceiling is too low).

## Platform event alerts

These fire on events rather than thresholds, and are on by default for every project:

- A deploy fails (build error or health-check timeout — see [Deployment errors](/troubleshooting/deployment-errors)).
- A service enters a restart loop (three crashes in five minutes).
- A cron run fails or overruns its schedule interval.
- A Postgres database approaches its storage limit (80%, then 95%).

## Notification channels

**Email** goes to chosen organization members. **Webhooks** POST a JSON payload to your URL:

```json
{
  "alert": "latency_p95 > 500ms for 5m",
  "state": "firing",
  "project": "my-app",
  "service": "api",
  "value": 743,
  "at": 1768650000
}
```

Webhook deliveries are signed with an `X-Nimbus-Signature` HMAC header (secret shown when the channel is created) and retried for one hour on non-2xx — verify like any webhook before trusting the payload.
