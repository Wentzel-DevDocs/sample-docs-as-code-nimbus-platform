---
title: nimbus logs
description: Stream, filter, and export logs for services, jobs, and builds - flags, retention, and structured output.
nav_order: 4
---
```bash
nimbus logs [flags]
```

Everything your instances write to stdout/stderr, plus edge request logs, in one stream.

## Flags

| Flag | Default | Description |
| --- | --- | --- |
| `--service <name>` | all | Limit to one service |
| `--job <name>` | - | Limit to a [job](/guides/compute/jobs)'s runs |
| `--build` | off | Show the most recent build's log instead |
| `--since <dur>` | `15m` | Look back, e.g. `1h`, `24h`, `7d` |
| `--level <lvl>` | all | `debug`, `info`, `warn`, or `error` and up |
| `--follow`, `-f` | off | Stream new lines as they arrive |
| `--instance <id>` | all | One instance (IDs from [`services status`](/reference/cli/nimbus-services)) |
| `--json` | off | One JSON object per line |

## Typical invocations

```bash
nimbus logs --service web --follow
nimbus logs --service web --since 1h --level error
nimbus logs --build                        # why did the image fail to build?
nimbus logs --service web --json | jq 'select(.request_id=="req_8f3k2")'
```

## Structure and levels

Lines your app emits as JSON are parsed - `level`, `msg`, and `request_id` become filterable fields; anything else rides along in `--json` output. Plain-text lines default to `info`. Edge request logs carry the same `X-Request-Id` the [load balancer](/guides/networking/load-balancing) attaches, so one ID follows a request from edge to app.

## Retention and export

Logs are retained for **14 days**. There is no long-term archive inside Nimbus - for longer retention, run `nimbus logs --json --since 24h` on a [schedule](/tutorials/scheduled-jobs) and ship the output to [object storage](/guides/data/object-storage) or your log system. High-volume services are sampled above 10k lines/minute per instance (a `sampled: true` field marks affected windows).
