---
title: Welcome to Nimbus
description: Nimbus is a cloud platform for deploying web services, workers, cron jobs, and static sites, with built-in object storage, a key-value store, and managed Postgres.
order: 0
status: stable
audience: all
category: overview
product: Nimbus Platform
owners: docs-writers
last_reviewed: 2025-10-14
---

# Welcome to Nimbus

Nimbus runs your applications so you do not have to run servers. You describe what a project needs in a single `nimbus.yaml` — services, schedules, storage — and Nimbus builds, deploys, scales, and monitors it across regions.

## What you can run

- **Web services** — HTTP apps built from a Dockerfile or a buildpack, with autoscaling and zero-downtime deploys.
- **Background workers** — long-running processes without an HTTP port.
- **Cron jobs** — commands on a schedule, down to one-minute granularity.
- **Static sites** — prebuilt assets served from the edge.

Alongside compute, every project can attach **Nimbus Store** (object storage), **Nimbus KV** (a low-latency key-value store), and **Nimbus Postgres** (managed PostgreSQL 16).

## The 60-second version

```bash
npm install -g @nimbus-platform/cli
nimbus login
nimbus init            # writes a starter nimbus.yaml
nimbus deploy          # builds and ships it
```

Your service comes up at `https://<service>-<project>.nimbusapp.example`, and `nimbus logs -f` tails it live.

## Find your way around

- New here? Start with [Install the CLI](/guides/getting-started/install-cli) and [Your first deployment](/guides/getting-started/first-deployment).
- Want the mental model? Read [Platform overview](/concepts/platform-overview), [Projects and environments](/concepts/projects-and-environments), and the [Glossary](/concepts/glossary).
- Building something concrete? The [tutorials](/tutorials/deploy-node-api) walk through complete apps.
- Looking something up? The [CLI](/reference/cli/overview), [REST API](/reference/api/overview), and [nimbus.yaml](/reference/config/nimbus-yaml) references cover every flag, endpoint, and key.
- Stuck? [Troubleshooting](/troubleshooting/build-failures) collects the failure modes we see most.
- Shipping changes? Check [Release notes](/release-notes).
