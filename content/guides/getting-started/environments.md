---
title: Environments
description: Use dev, staging, and production environments, promote a commit between them, and manage per-environment config.
nav_order: 3
---
Every project has isolated environments - `dev`, `staging`, and `production` by default, plus any you create. What "isolated" means precisely is covered in [Projects and organizations](/concepts/projects-and-orgs); this page is the workflow.

## Target an environment

Every command takes `--env`, or set a default with `nimbus config set env dev`:

```bash
nimbus deploy --env staging
nimbus logs --env production --service web
```

## Promote between environments

There is no separate "promote" verb: promotion is deploying the same commit to the next environment.

```bash
nimbus deploy --env staging          # verify here
nimbus deploy --env production       # then ship the same commit
```

Because the [image is content-addressed](/concepts/architecture), deploying an already-built commit to another environment skips the build and reuses the image - production gets byte-for-byte what staging ran.

## Per-environment configuration

`nimbus.yaml` is shared; differences live in environment variables and secrets, which are set per environment:

```bash
nimbus env set LOG_LEVEL=debug --env dev
nimbus secrets set DATABASE_URL --env production   # value read from stdin
```

Precedence and `${VAR}` interpolation rules are in [Environment variables](/reference/config/environment-variables). Attached data follows the same rule: a [Postgres](/guides/data/managed-postgres) instance attached in staging is invisible to production.

## Ephemeral environments

Custom environments are cheap - a common pattern is one per pull request:

```bash
nimbus envs create preview-142
nimbus deploy --env preview-142
# ... review ...
nimbus envs delete preview-142     # tears down everything in it
```

Deleting an environment stops its workloads and deletes its data stores after a confirmation prompt. Combine with [scale-to-zero](/guides/compute/scaling) to keep idle previews free.
