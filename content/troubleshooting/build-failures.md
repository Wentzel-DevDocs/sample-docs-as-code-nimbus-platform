---
title: Build failures
description: Diagnosing failed builds — reading build logs, dependency and lockfile errors, buildpack detection failures, timeouts, and oversized images.
order: 1
status: stable
audience: all
category: troubleshooting
product: Nimbus Platform
owners: docs-writers
last_reviewed: 2026-07-01
---

# Build failures

A deploy that dies in the `building` phase never touched your running service — fix the build and redeploy. Start by reading the build log:

```bash
nimbus logs <service> --deploy <dep_id>
```

The failing step is the last one printed. The patterns below cover most failures.

## Dependency install fails

`npm ci` (and its Python/Ruby equivalents) fails when the lockfile and manifest disagree, or when a dependency needs system libraries your image lacks.

- `npm ci can only install with an existing package-lock.json` — commit the lockfile.
- `EINTEGRITY` / checksum mismatches — regenerate the lockfile locally and commit.
- `node-gyp` errors — a native module needs build tools; add them in the Dockerfile (`apt-get install -y python3 make g++`) or switch to a `-slim` rather than `-alpine` base image.

## Buildpack cannot detect the app

`buildpack: node` requires a `package.json` at the build context root (equivalents for `python`, `go`, `ruby`). If your app lives in a subdirectory, set `build.context: apps/api` in [nimbus.yaml](/reference/config/nimbus-yaml) — or write a Dockerfile, which always wins for unusual layouts.

## Dockerfile not found

`build.dockerfile` is resolved relative to `build.context`. A monorepo where the Dockerfile sits beside the service usually wants:

```yaml
    build:
      context: apps/api
      dockerfile: Dockerfile
```

## Build timeout (30 minutes)

Almost always a caching problem: copying the whole source **before** installing dependencies invalidates the install layer on every commit. Copy manifests first, install, then copy source — the pattern shown in the [Node tutorial](/tutorials/deploy-node-api). Genuine 30-minute builds should move heavy artifact generation into CI and `COPY` the results.

## Image too large (8 GiB)

Use multi-stage builds so compilers and dev dependencies stay out of the final stage, add a `.dockerignore` (`node_modules`, `.git`, test fixtures), and prefer `-slim` bases.

## It builds locally but not on Nimbus

Nimbus builds `linux/amd64` from a clean context. The usual culprits: an arm64-only base image tag, files present locally but ignored by `.gitignore`/`.nimbusignore` (they are excluded from upload — see [the deploy reference](/reference/cli/deploy)), or reliance on host networks. If the build passed but the rollout failed, that is the next page: [Deployment errors](/troubleshooting/deployment-errors).
