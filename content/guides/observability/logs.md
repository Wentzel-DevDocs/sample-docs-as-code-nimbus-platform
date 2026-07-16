---
title: Logs
description: Where service logs go, tailing and filtering with nimbus logs, structured logging, retention, and shipping logs elsewhere.
order: 1
status: stable
audience: developers
category: how-to
product: Nimbus Platform
owners: docs-writers
last_reviewed: 2026-07-16
---

# Logs

Anything a service writes to stdout or stderr is collected, timestamped, tagged with service, instance, and deploy, and kept searchable. No agent, no configuration.

## Tail and filter

```bash
nimbus logs api -f                       # follow live
nimbus logs api --since 1h              # recent window
nimbus logs api --grep "status=5"       # substring/regex filter
nimbus logs api --deploy dep_8c31f2      # logs of one deploy (includes build output)
nimbus logs reports --run latest         # one cron run
```

The dashboard offers the same filters with a time-range picker, plus full-text search across services.

## Write structured logs

Emit one JSON object per line and the platform indexes the fields:

```json
{"level":"error","msg":"charge failed","request_id":"req_9a1b","order":8812,"status":502}
```

Include the `X-Request-Id` header value (see [Networking](/concepts/networking)) in every line a request produces — `nimbus logs api --grep req_9a1b` then reconstructs a request across services.

## What lands in logs automatically

- Router access lines for every HTTP request (method, path, status, duration, request id).
- Build output, attached to the deploy.
- Instance lifecycle events: starts, health-check failures, `SIGTERM`, OOM kills.

Lifecycle lines are the first thing to read when [troubleshooting deploys](/troubleshooting/deployment-errors).

## Retention and limits

| Plan | Retention |
| --- | --- |
| Free | 7 days |
| Pro | 30 days |
| Business | 90 days |

Lines are capped at 16 KB (longer lines are truncated, not dropped). Need logs elsewhere? Create a **log drain** under **Project → Observability** to stream everything to an HTTPS endpoint you run — the payload is newline-delimited JSON, batched per second.

Logs answer "what happened"; [metrics](/guides/observability/metrics) answer "how much, how fast".
