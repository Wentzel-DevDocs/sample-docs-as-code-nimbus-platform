---
title: Build an app on Postgres
description: Attach Nimbus Postgres to a service, run schema migrations safely, query from the app, seed preview environments, and test a restore.
order: 4
status: stable
audience: developers
category: tutorial
product: Nimbus Platform
owners: docs-writers
last_reviewed: 2026-02-10
---

# Build an app on Postgres

This tutorial extends the [Node.js API tutorial](/tutorials/deploy-node-api) with a real database: attach it, migrate it, query it, and prove you can restore it.

## 1. Attach the database

```yaml
resources:
  - name: maindb
    type: postgres
    plan: standard-1
    per_environment: true
```

Deploying provisions the database and injects `NIMBUS_DATABASE_URL` into every service. `per_environment: true` gives each [preview](/concepts/projects-and-environments) its own empty copy, so branches cannot corrupt production data.

## 2. Run migrations as a release step

Do not migrate at process boot — ten instances booting would race. Run migrations once per release with `nimbus run`, which executes a one-off command in the service's built image with the same environment:

```bash
nimbus deploy api
nimbus run api -- npm run migrate
```

In CI, those two lines are the whole release script. The reasoning is covered in [Managed Postgres](/guides/data/nimbus-postgres).

## 3. Query from the app

```js
import pg from "pg";
const pool = new pg.Pool({ connectionString: process.env.NIMBUS_DATABASE_URL, max: 8 });

app.get("/api/quotes", async (req, res) => {
  const { rows } = await pool.query("select id, text from quotes order by id desc limit 20");
  res.json({ quotes: rows });
});
```

Keep `max` modest — platform-side pooling plus [autoscaling](/guides/compute/scaling) means instance count × pool size must stay under Postgres's connection slots.

## 4. Seed preview environments

Previews start empty. Add a tiny seed script and run it against the preview:

```bash
nimbus run api --env preview/feature-checkout -- npm run seed
```

## 5. Prove the restore path

Before you need it in anger:

```bash
nimbus db restore maindb --at "2026-07-15T09:30:00Z" --into maindb-drill
nimbus db connect maindb-drill      # poke around, confirm the data
```

Restores create a new resource and never touch the original — details in [Managed Postgres](/guides/data/nimbus-postgres). Delete the drill copy when done by removing it from `nimbus.yaml` and deploying.
