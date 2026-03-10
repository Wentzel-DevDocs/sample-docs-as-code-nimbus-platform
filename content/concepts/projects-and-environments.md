---
title: Projects and environments
description: What a Nimbus project owns, how production and preview environments work, and how configuration varies per environment.
order: 2
status: stable
audience: all
category: concept
product: Nimbus Platform
owners: docs-writers
last_reviewed: 2026-03-10
---

# Projects and environments

A **project** is one application: a `nimbus.yaml`, a primary region, a set of services, and its attached resources. An **environment** is an isolated copy of that project's services and configuration.

## Environments you get by default

- `production` — created with the project; the environment `nimbus deploy` targets by default.
- Preview environments — created automatically per branch when a git repository is connected, named after the branch, and deleted when the branch is merged or deleted.

You can also create long-lived custom environments (for example `staging`) with `nimbus env create staging`.

## What is isolated per environment

Each environment has its own service instances, URLs, [secrets](/reference/cli/secrets), logs, and metrics. Preview URLs look like `https://<service>-<project>-<branch>.nimbusapp.example`.

Resources are **shared by default** — a preview points at the same Postgres database as production unless you mark the resource `per_environment: true` in [nimbus.yaml](/reference/config/nimbus-yaml), which provisions a fresh copy per environment (empty database, empty bucket).

## Targeting an environment

Most CLI commands take `--env`:

```bash
nimbus deploy --env staging
nimbus logs api --env preview/feature-checkout -f
nimbus secrets set STRIPE_KEY=sk_test_123 --env staging
```

## Promotion pattern

A common flow: deploy to `staging`, verify, then deploy the same commit to production. Because builds are keyed on the commit, the production deploy reuses the already-built image rather than rebuilding — see [Deploying services](/guides/compute/deploying-services).

## Deleting

`nimbus env delete staging` tears down an environment's services and any `per_environment` resources. The production environment cannot be deleted separately from the project.
