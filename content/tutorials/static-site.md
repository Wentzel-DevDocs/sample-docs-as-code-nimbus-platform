---
title: Deploy a static site
description: Build and deploy a static frontend to the edge — the static service type, build command, publish directory, caching, and a custom domain.
order: 2
status: stable
audience: developers
category: tutorial
product: Nimbus Platform
owners: docs-writers
last_reviewed: 2026-01-13
---

# Deploy a static site

Static services upload a built directory and serve it from the edge in every [region](/concepts/regions-and-edge) — no instances, no scaling to think about. This tutorial deploys a Vite app; any generator that emits a directory works identically.

## 1. Declare the service

```yaml
name: marketing
region: fra
services:
  - name: site
    type: static
    build:
      buildpack: node
      command: npm run build
    publish_dir: dist
```

At deploy time Nimbus runs `npm ci`, then your `build.command`, then uploads `publish_dir`. Nothing runs after the build — a static service has no process.

## 2. Deploy

```bash
nimbus deploy site
# dep_2210af  building    npm run build
# dep_2210af  live        https://site-marketing.nimbusapp.example
```

## 3. How requests are served

- Files are served from the nearest edge location worldwide.
- Fingerprinted assets (`app.3f9c1a.js`) get `Cache-Control: immutable` for a year automatically.
- HTML gets `no-cache` so a new deploy is visible immediately.
- A request for a path without an extension falls back to `/index.html`, which is what single-page-app routers need. Put a `404.html` in the publish dir for a custom not-found page.

## 4. Add your domain

```yaml
    domains:
      - www.example.com
```

Deploy, add the CNAME, and TLS is issued automatically — full details in [Custom domains](/guides/compute/custom-domains).

## Pairing with an API

A common layout is this static site plus a `web` service in the same project; the frontend calls the API by its public URL or custom domain. Same-project services also share [preview environments](/concepts/projects-and-environments), so every branch gets a coherent frontend + API preview pair.
