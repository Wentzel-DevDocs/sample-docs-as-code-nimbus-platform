---
title: Environment variables
description: The three sources of environment variables — nimbus.yaml env, secrets, and platform-injected NIMBUS_* variables — and their precedence.
order: 2
status: stable
audience: developers
category: reference
product: Nimbus Platform
owners: docs-writers
last_reviewed: 2025-12-09
---

# Environment variables

Service processes see environment variables from three sources, merged at instance start.

## 1. Plain configuration: `env` in nimbus.yaml

```yaml
services:
  - name: api
    env:
      LOG_LEVEL: info
      FEATURE_CHECKOUT_V2: "true"
```

These are code-reviewed with the rest of [nimbus.yaml](/reference/config/nimbus-yaml) and identical across environments. Values are strings; quote anything YAML would otherwise coerce.

## 2. Secrets

Anything sensitive is set with [`nimbus secrets`](/reference/cli/secrets), per environment, encrypted, and write-only. Changing a secret rolls the environment's services.

## 3. Platform-injected `NIMBUS_*` variables

Injected automatically; the `NIMBUS_` prefix is reserved.

| Variable | Contents |
| --- | --- |
| `NIMBUS_ENV` | Environment name, such as `production` or `preview/feature-x` |
| `NIMBUS_REGION` | The project [region](/concepts/regions-and-edge) |
| `NIMBUS_SERVICE` | This service's name |
| `NIMBUS_DEPLOYMENT` | The live `dep_...` ID |
| `NIMBUS_DATABASE_URL` | Connection string, per attached [Postgres](/guides/data/nimbus-postgres) resource |
| `NIMBUS_STORE_<NAME>_URL` / `_KEY` | Per attached [Store bucket](/guides/data/object-storage) |
| `NIMBUS_KV_<NAME>_URL` / `_KEY` | Per attached [KV namespace](/guides/data/key-value-store) |

Resource `<NAME>` is the resource name uppercased, hyphens as underscores.

## Precedence

For a variable defined in several places, highest wins:

1. Service-scoped secret
2. Project secret
3. `env:` in `nimbus.yaml`

Platform `NIMBUS_*` variables cannot be overridden.

## Practical rules

- Branch on `NIMBUS_ENV` for environment-specific behavior instead of maintaining parallel config files.
- Total environment size per service is capped at 32 KB — large blobs belong in [Store](/guides/data/object-storage), fetched at boot.
- Never log the full environment; secret values look exactly like plain ones once injected.
