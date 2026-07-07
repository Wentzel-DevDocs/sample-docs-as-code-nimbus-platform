---
title: nimbus deploy
description: Full reference for nimbus deploy — arguments, flags, what gets uploaded, build behavior, and CI usage.
order: 2
status: stable
audience: developers
category: reference
product: Nimbus Platform
owners: docs-writers
last_reviewed: 2026-07-07
---

# nimbus deploy

Builds and rolls out services, streaming progress until the deploy is `live` or `failed`. The lifecycle it drives is described in [Deploying services](/guides/compute/deploying-services).

## Usage

```bash
nimbus deploy [services...] [flags]
```

With no arguments, every service whose inputs changed since its last deploy is rebuilt and rolled out. Naming services restricts the deploy to them.

## Flags

| Flag | Default | Description |
| --- | --- | --- |
| `--env <name>` | `production` | Target [environment](/concepts/projects-and-environments) |
| `--message <text>` | — | Note attached to the deploy, shown in history |
| `--no-wait` | wait | Return after the build is queued instead of streaming to completion |
| `--force` | off | Rebuild even when inputs are unchanged |
| `--dry-run` | off | Validate `nimbus.yaml` and print the plan without deploying |

## What gets uploaded

The current directory, minus `.gitignore` matches and anything in an optional `.nimbusignore`. In a git repository, the commit SHA is recorded on the deploy; builds are cached per commit, which is what makes same-commit promotion between environments skip the build.

## Exit behavior

Streaming mode exits `0` only when every named service reaches `live`. A health-check failure prints the failing instances' last log lines (the same ones `nimbus logs --deploy <id>` shows) and exits `1`, leaving the previous version serving.

## In CI

```bash
export NIMBUS_TOKEN=$CI_SECRET_NIMBUS_TOKEN
nimbus deploy --env production --message "$CI_COMMIT_TITLE" --json
```

`--json` emits one status object per line, ending with the final deploy record — parse the last line for the deploy ID and result. Common failures and their fixes: [Build failures](/troubleshooting/build-failures), [Deployment errors](/troubleshooting/deployment-errors).
