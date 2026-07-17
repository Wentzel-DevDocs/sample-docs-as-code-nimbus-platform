---
title: Your first deploy
description: Initialize nimbus.yaml, deploy a service, watch the release go live, and roll it back - end to end in a few minutes.
nav_order: 2
---
This walkthrough takes an app from a local directory to a live URL. It assumes the [CLI is installed](/guides/getting-started/install-cli) and you are signed in.

## 1. Initialize

From your app's directory:

```bash
nimbus init
```

`init` detects the runtime (a `package.json`, `go.mod`, `Dockerfile`, and so on), asks for a service name and [region](/concepts/regions-and-zones), and writes a starter [`nimbus.yaml`](/reference/config/nimbus-yaml):

```yaml
service: web
region: us-east
size: small
build:
  builder: auto
ports:
  - 8080
healthcheck:
  path: /healthz
```

Your app must listen on the port it declares and answer the health check path with a 2xx.

## 2. Deploy

```bash
nimbus deploy
```

Deploy uploads the source, builds an image, starts new instances, waits for health checks, and only then shifts traffic:

```text
==> Building web (auto: node@20)
==> Pushing image
==> Starting 2 instances in us-east (zones a, b)
==> Health checks passing (2/2)
==> Release r-1 is live
    https://web-storefront.nimbusapp.example
```

Every service gets a `https://<service>-<project>.nimbusapp.example` URL with TLS. If the build or health check fails, traffic never moves and `deploy` exits non-zero - see [deploy failures](/troubleshooting/deploy-failures) and the [exit codes](/reference/cli/nimbus-deploy).

## 3. Look around

```bash
nimbus services list          # what is running
nimbus logs --service web --follow
nimbus releases list          # r-1 and its predecessors
```

## 4. Roll back (when you need to)

Releases are immutable, so rollback is instant:

```bash
nimbus releases rollback r-1
```

## Next steps

- Add `staging` and `production` targets: [Environments](/guides/getting-started/environments).
- Put your own hostname on it: [Custom domains](/guides/networking/domains).
- Follow a complete example: [Deploy a Node.js app](/tutorials/deploy-node-app).
