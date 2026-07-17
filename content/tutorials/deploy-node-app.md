---
title: Deploy a Node.js app
description: Take an Express app with a Postgres database from laptop to production - init, database, migrations, deploy, and CI.
nav_order: 1
---
End to end: an Express app with a database, deployed to production with CI publishing on merge. Have the [CLI installed](/guides/getting-started/install-cli) first.

## 1. Make the app deployable

Nimbus needs two things from a Node app: listen on `PORT`, and answer a health check.

```js
import express from "express";
const app = express();

app.get("/healthz", (req, res) => res.sendStatus(204));
app.get("/", (req, res) => res.send("hello from nimbus"));

app.listen(process.env.PORT || 8080);
```

## 2. Initialize

```bash
nimbus init
```

Pick the service name `web` and a [region](/concepts/regions-and-zones). The generated [`nimbus.yaml`](/reference/config/nimbus-yaml) detects `builder: auto` from `package.json`; set `healthcheck.path: /healthz`.

## 3. Add Postgres

```bash
nimbus pg create maindb --plan dev --env dev
nimbus pg attach maindb --service web --env dev
```

The app now receives `DATABASE_URL` [automatically](/guides/data/managed-postgres). Run migrations as a [one-off job](/guides/compute/jobs) so they execute in the same network with the same variables:

```bash
nimbus jobs run --env dev -- npx node-pg-migrate up
```

## 4. Deploy to dev, then promote

```bash
nimbus deploy --env dev
# smoke test the printed URL, then:
nimbus pg create maindb --plan standard-2 --env production
nimbus pg attach maindb --service web --env production
nimbus jobs run --env production -- npx node-pg-migrate up
nimbus deploy --env production
```

[Promotion](/guides/getting-started/environments) reuses the built image - production runs the exact bytes dev verified.

## 5. Publish from CI

Create a [service token](/concepts/identity-and-access) and add it to your CI secrets as `NIMBUS_TOKEN`:

```yaml
# .github/workflows/deploy.yml
on:
  push:
    branches: [main]
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: curl -fsSL https://get.nimbus.example/install.sh | sh
      - run: nimbus deploy --project storefront --env production
        env:
          NIMBUS_TOKEN: ${{ secrets.NIMBUS_TOKEN }}
```

The [deploy exit codes](/reference/cli/nimbus-deploy) make the CI gate honest: a failed health check fails the workflow, and traffic never moved. From here: [scaling](/guides/compute/scaling) for real load, and a [custom domain](/tutorials/custom-domain-tls) to finish the job.
