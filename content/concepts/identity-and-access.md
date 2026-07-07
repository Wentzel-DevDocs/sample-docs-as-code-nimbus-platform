---
title: Identity and access
description: Organization roles, what each role can do, personal access tokens for the API, and how secrets are protected.
order: 6
status: stable
audience: all
category: concept
product: Nimbus Platform
owners: docs-writers
last_reviewed: 2026-07-07
---

# Identity and access

Access in Nimbus is governed at the organization level through roles, and programmatic access happens through personal access tokens.

## Roles

| Role | Can |
| --- | --- |
| `viewer` | Read dashboards, logs, and metrics |
| `developer` | Everything above, plus deploy, manage secrets, and create preview environments |
| `admin` | Everything above, plus manage projects, custom domains, resources, and members (except owners) |
| `owner` | Everything, including billing and deleting the organization |

Roles apply across every project in the organization; per-project roles are not supported today. Invite members under **Organization → Members**.

## Personal access tokens

The [REST API](/reference/api/overview) and CI systems authenticate with personal access tokens, prefixed `nbp_`. Create them under **Account → Tokens**, where each token can be scoped to `read`, `deploy`, or `full` and given an expiry. A token acts with its creator's role, capped by its scope.

```bash
curl https://api.nimbus.example/v1/projects \
  -H "Authorization: Bearer nbp_2f8a1example"
```

Tokens are shown once at creation. Revoke leaked tokens immediately under the same screen; revocation is instant.

## The CLI's credentials

`nimbus login` opens a browser flow and stores a token in your OS keychain — the CLI never writes plaintext credentials to disk. `nimbus logout` removes it. In CI, skip the browser flow by setting the `NIMBUS_TOKEN` environment variable, as shown in [Install the CLI](/guides/getting-started/install-cli).

## How secrets are protected

[Secrets](/reference/cli/secrets) are encrypted at rest, never returned by any API once written, and injected into service instances as environment variables at start time. `developer` and above can set them; nobody can read them back — rotate rather than recover.
