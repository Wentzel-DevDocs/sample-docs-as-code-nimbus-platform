---
title: CLI overview
description: The nimbus CLI at a glance — every command grouped by task, global flags, environment variables, and exit codes.
order: 1
status: stable
audience: developers
category: reference
product: Nimbus Platform
owners: docs-writers
last_reviewed: 2025-12-09
---

# CLI overview

The `nimbus` CLI drives the whole platform from a terminal. Install and authentication are covered in [Install the CLI](/guides/getting-started/install-cli); this page is the map of every command.

## Commands by task

| Command | Purpose |
| --- | --- |
| `nimbus init` | Write a starter `nimbus.yaml` |
| `nimbus deploy [services...]` | Build and roll out — see [deploy](/reference/cli/deploy) |
| `nimbus status` | Live deploy, instances, and health per service |
| `nimbus logs <service>` | Tail and filter logs — see [logs](/reference/cli/logs) |
| `nimbus rollback [deploy-id]` | Re-point to a previous deploy |
| `nimbus secrets <set,list,rm>` | Manage secrets — see [secrets](/reference/cli/secrets) |
| `nimbus env <create,list,delete>` | Manage [environments](/concepts/projects-and-environments) |
| `nimbus domains <add,list,rm>` | Manage [custom domains](/guides/compute/custom-domains) |
| `nimbus db <connect,url,restore>` | Work with [Postgres](/guides/data/nimbus-postgres) |
| `nimbus store sync <dir> <bucket>` | Bulk upload to [Store](/guides/data/object-storage) |
| `nimbus run <service> -- <cmd>` | Run a one-off command in a service's image |
| `nimbus open` | Open the service URL in a browser |
| `nimbus whoami`, `nimbus login`, `nimbus logout` | Session management |
| `nimbus version`, `nimbus upgrade` | CLI maintenance |

## Global flags

| Flag | Meaning |
| --- | --- |
| `--env <name>` | Target environment (default `production`) |
| `--project <name>` | Override project detection from `nimbus.yaml` |
| `--json` | Machine-readable output for scripting |
| `--quiet` | Errors only |

## Environment variables

| Variable | Effect |
| --- | --- |
| `NIMBUS_TOKEN` | Token auth, bypassing the keychain — the CI pattern |
| `NIMBUS_ENV` | Default for `--env` |
| `NO_COLOR` | Disable ANSI color |

## Exit codes

`0` success, `1` command error (bad flags, failed deploy), `2` authentication problem, `3` network problem. CI can branch on these; with `--json`, stdout stays parseable while progress goes to stderr.
