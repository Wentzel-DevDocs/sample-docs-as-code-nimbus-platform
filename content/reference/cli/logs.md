---
title: nimbus logs
description: Full reference for nimbus logs — following, time windows, grep filters, deploy and cron-run selectors, and JSON output.
order: 3
status: stable
audience: developers
category: reference
product: Nimbus Platform
owners: docs-writers
last_reviewed: 2026-03-10
---

# nimbus logs

Fetches or follows a service's logs. Concepts, structured logging, and retention live in the [logs guide](/guides/observability/logs); this page is the flag reference.

## Usage

```bash
nimbus logs <service> [flags]
```

## Flags

| Flag | Default | Description |
| --- | --- | --- |
| `-f, --follow` | off | Stream new lines as they arrive |
| `--since <dur>` | `15m` | Window start: `30s`, `15m`, `6h`, `3d`, or an RFC 3339 timestamp |
| `--until <time>` | now | Window end (RFC 3339) |
| `--grep <pattern>` | — | Only lines matching the regex |
| `--instance <id>` | all | One instance only |
| `--deploy <id>` | — | Logs of one deploy, **including its build output** |
| `--run <id,latest>` | — | One cron run (cron services only) |
| `--env <name>` | `production` | Target environment |
| `--json` | off | Raw JSON lines with all metadata fields |

## Examples

```bash
nimbus logs api -f --grep "level\":\"error"     # follow errors on a JSON-logging service
nimbus logs api --since 2026-07-15T09:00:00Z --until 2026-07-15T10:00:00Z
nimbus logs api --deploy dep_8c31f2              # why did that deploy fail?
nimbus logs reports --run latest                 # last cron run, start to finish
nimbus logs api --grep req_9a1b                  # one request across its lifetime
```

## Output format

Human mode prints `time  instance  line`. With `--json`, each line is an object: `{ "time", "service", "instance", "deploy", "level", "line" }` — `level` is populated when the service logs structured JSON. Piping into `jq` composes naturally with `--json`.
