---
title: Deploy a static site
description: Ship a prebuilt static site - or a generator like Vite - with the static builder, SPA fallback, and cache headers.
nav_order: 2
---
Static sites use `builder: static`: no server process, no `PORT`, no health check to write - Nimbus serves the files from its [edge](/guides/networking/load-balancing).

## 1. Configure

For a site that is already built into `dist/`:

```yaml
service: site
region: eu-west
build:
  builder: static
  dir: dist
```

If a build step produces `dist/`, declare it and Nimbus runs it during [deploy](/reference/cli/nimbus-deploy):

```yaml
build:
  builder: static
  command: ["npm", "run", "build"]   # runs with Node available
  dir: dist
```

## 2. Deploy

```bash
nimbus deploy
```

```text
==> Building site (static: npm run build)
==> Uploading 214 files (3.8 MiB)
==> Release r-1 is live
    https://site-storefront.nimbusapp.example
```

Releases and [rollback](/guides/compute/services) work exactly as for services - a static release is an immutable snapshot of the files.

## 3. Single-page apps

Client-side routers need unknown paths to fall back to `index.html`:

```yaml
build:
  builder: static
  dir: dist
  spa: true
```

Real files still win over the fallback, and missing *assets* (anything with an extension) still 404 - only extension-less routes fall back.

## 4. Headers and caching

Hashed assets are cached hard; HTML is not:

- Files matching `*.<hash>.*` (e.g. `app.9f3d2c.js`) -> `Cache-Control: public, max-age=31536000, immutable`
- Everything else -> `Cache-Control: public, max-age=60`

Override per path if needed:

```yaml
build:
  builder: static
  dir: dist
  headers:
    - path: "/fonts/*"
      cache_control: "public, max-age=604800"
```

## 5. Finish

Static services scale automatically at the edge - there is no `scaling:` block to tune and no instances to see in [`services status`](/reference/cli/nimbus-services). Add your own hostname with the [custom domain tutorial](/tutorials/custom-domain-tls); it works identically for static services.
