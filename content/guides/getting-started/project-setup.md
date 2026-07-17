---
title: Set up a project
description: Create a Nimbus project with nimbus init, understand the starter nimbus.yaml, pick a region, and optionally connect a git repository.
order: 2
status: stable
audience: developers
category: how-to
product: Nimbus Platform
owners: docs-writers
last_reviewed: 2026-07-01
---

# Set up a project

A project is born from a `nimbus.yaml` at the root of your repository. `nimbus init` writes a sensible starter for you.

## Initialize

From your application's directory:

```bash
nimbus init
```

The CLI asks for a project name and a [region](/concepts/regions-and-edge), inspects the directory (a `Dockerfile`, a `package.json`, or a static `dist/`), and writes a matching starter:

```yaml
name: my-app
region: iad
services:
  - name: web
    type: web
    build:
      dockerfile: Dockerfile
    http:
      port: 8080
      health_check: /healthz
```

Every key is documented in the [nimbus.yaml reference](/reference/config/nimbus-yaml). Commit the file — it is the single source of truth for how the project runs.

## No Dockerfile? Use a buildpack

If the directory has no Dockerfile, `nimbus init` configures a buildpack instead:

```yaml
    build:
      buildpack: node
```

Buildpacks exist for `node`, `python`, `go`, and `ruby`; they detect the standard entry points (`npm start`, `gunicorn`, a `main` package, and so on). A Dockerfile always wins when you need full control.

## Add a health check endpoint

Web services must answer their `health_check` path with a `2xx` once ready. A one-line handler is enough:

```js
app.get("/healthz", (req, res) => res.send("ok"));
```

Deploys gate on this — see [Services](/concepts/services).

## Connect a git repository (optional)

Under **Project → Git** in the dashboard, connect a repository to get push-to-deploy: pushes to the default branch deploy `production`, and every other branch gets a [preview environment](/concepts/projects-and-environments). You can keep deploying from the CLI as well; the flows coexist.

Next: [your first deployment](/guides/getting-started/first-deployment).
