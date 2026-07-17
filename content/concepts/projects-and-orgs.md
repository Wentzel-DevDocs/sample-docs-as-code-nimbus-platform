---
title: Projects and organizations
description: The org -> project -> environment hierarchy, what is isolated per environment, and how the CLI targets them.
nav_order: 3
---
Everything in Nimbus hangs off a three-level hierarchy:

```text
Organization
  Project
    Environment (dev, staging, production, ...)
      services, jobs, databases, buckets, ...
```

## Organizations

The billing and membership boundary. [Members and roles](/concepts/identity-and-access) are managed at the org level, and [usage](/concepts/billing) rolls up to one invoice per org. Most teams have exactly one.

## Projects

A project groups the workloads of one application or system - say `storefront` with its web service, worker, and database. Quotas (number of services, domains, and so on - defaults in the [FAQ](/troubleshooting/faq)) apply per project.

## Environments

Each project starts with three environments - `dev`, `staging`, and `production` - and you can add custom ones (`nimbus envs create preview-123`). An environment is a full isolation boundary:

- Separate instances, [private network](/guides/networking/private-networks), and internal DNS.
- Separate [environment variables and secrets](/reference/config/environment-variables).
- Separate attached data: staging talks to the staging database, never production's.

The same `nimbus.yaml` deploys to any environment; only the target changes. Promotion is redeploying the same commit to the next environment - see [Environments](/guides/getting-started/environments).

## Targeting from the CLI

Every command accepts `--project` and `--env`, with defaults from your [CLI configuration](/reference/cli/overview):

```bash
nimbus deploy --project storefront --env staging
nimbus logs --project storefront --env production --service web
```
