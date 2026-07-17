---
title: Deploy a Node.js API
description: A complete walkthrough deploying an Express API — Dockerfile, health check, nimbus.yaml, first deploy, secrets, and autoscaling.
order: 1
status: stable
audience: developers
category: tutorial
product: Nimbus Platform
owners: docs-writers
last_reviewed: 2026-07-01
---

# Deploy a Node.js API

This tutorial ships a small Express API from an empty directory to a scaled, secret-carrying production service. Prerequisites: the [CLI installed](/guides/getting-started/install-cli) and `nimbus login` done.

## 1. The application

```js
// server.js
import express from "express";
const app = express();

app.get("/healthz", (req, res) => res.send("ok"));
app.get("/api/quotes", (req, res) => {
  res.json({ quotes: ["Ship it.", "Boring is good."], env: process.env.NIMBUS_ENV });
});

app.listen(process.env.PORT || 8080, "0.0.0.0", () =>
  console.log(JSON.stringify({ level: "info", msg: "listening", port: 8080 }))
);
```

Two details matter to Nimbus: the `/healthz` endpoint (deploys gate on it — see [Services](/concepts/services)) and binding to `0.0.0.0`, not `localhost`.

## 2. The Dockerfile

```dockerfile
FROM node:20-slim
WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev
COPY . .
EXPOSE 8080
CMD ["node", "server.js"]
```

Copying `package*.json` before the source keeps the `npm ci` layer cached between deploys (see [Deploying services](/guides/compute/deploying-services)).

## 3. nimbus.yaml

```yaml
name: quotes-api
region: iad
services:
  - name: api
    type: web
    build: { dockerfile: Dockerfile }
    http: { port: 8080, health_check: /healthz }
    scale: { min: 1, max: 4, target_concurrency: 40 }
    size: micro
```

`target_concurrency` fits an I/O-light API better than CPU — the reasoning is in [Scaling](/guides/compute/scaling).

## 4. Deploy and verify

```bash
nimbus deploy
curl https://api-quotes-api.nimbusapp.example/api/quotes
nimbus logs api -f
```

## 5. Add a secret

```bash
nimbus secrets set QUOTES_SIGNING_KEY=$(openssl rand -hex 32)
```

The set triggers a rolling restart; the process then reads `process.env.QUOTES_SIGNING_KEY`. Precedence rules are in [Environment variables](/reference/config/environment-variables).

## Where to go from here

Attach [Postgres](/tutorials/postgres-app) for real data, add a [custom domain](/guides/compute/custom-domains), and set the starter [alerts](/guides/observability/alerts).
