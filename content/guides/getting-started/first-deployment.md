---
title: Your first deployment
description: Run nimbus deploy, follow the build and health-check phases, verify the live URL, and learn how rollback works.
order: 3
status: stable
audience: developers
category: how-to
product: Nimbus Platform
owners: docs-writers
last_reviewed: 2025-12-09
---

# Your first deployment

With the [CLI installed](/guides/getting-started/install-cli) and a [project set up](/guides/getting-started/project-setup), shipping is one command.

## Deploy

```bash
nimbus deploy
```

The CLI uploads your source, then streams the build and rollout:

```text
dep_8c31f2  queued
dep_8c31f2  building    step 4/9 RUN npm ci
dep_8c31f2  deploying   waiting for health check on /healthz
dep_8c31f2  live        https://web-my-app.nimbusapp.example
```

A deploy moves through `queued → building → deploying → live`. The `deploying` phase starts new instances and polls the health check; traffic only cuts over once it passes, so a broken build never takes down the running version.

## Verify

```bash
curl https://web-my-app.nimbusapp.example/healthz
# ok
nimbus status
# web  live  dep_8c31f2  2 instances  region iad
```

`nimbus open` opens the service URL in your browser.

## Watch it run

```bash
nimbus logs web -f
```

Structured log lines include the `X-Request-Id` from [networking](/concepts/networking), which makes tracing a single request trivial. More in the [logs guide](/guides/observability/logs).

## When a deploy fails

A failed build or health check leaves the previous version serving and marks the deploy `failed`. Read the streamed error, or re-fetch it later with `nimbus logs --deploy dep_8c31f2`. The common causes and fixes are collected in [Build failures](/troubleshooting/build-failures) and [Deployment errors](/troubleshooting/deployment-errors).

## Roll back

Every deploy is immutable, so rollback is just re-pointing:

```bash
nimbus rollback            # to the previous live deploy
nimbus rollback dep_5a09e1 # to a specific one
```

Rollbacks skip the build phase and complete in seconds.

## Where to next

Add [scaling](/guides/compute/scaling), attach a [database](/guides/data/nimbus-postgres), or put the app behind a [custom domain](/guides/compute/custom-domains).
