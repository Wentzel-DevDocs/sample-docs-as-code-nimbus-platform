---
title: API authentication
description: Personal access tokens for the REST API — creating and scoping them, sending the Authorization header, expiry, revocation, and CI patterns.
order: 2
status: stable
audience: developers
category: reference
product: Nimbus Platform
owners: docs-writers
last_reviewed: 2026-07-01
---

# API authentication

Every API request carries a personal access token (PAT) as a bearer credential. Tokens act with their creator's [organization role](/concepts/identity-and-access), further capped by the token's scope.

## Create a token

Under **Account → Tokens** in the dashboard: name it, pick a scope, optionally set an expiry. The value — prefixed `nbp_` — is shown **once**.

| Scope | Allows |
| --- | --- |
| `read` | All `GET` endpoints |
| `deploy` | `read` plus creating deployments and managing secrets |
| `full` | Everything the creator's role allows |

Give automation the narrowest scope that works: a status dashboard needs `read`; a deploy bot needs `deploy`; almost nothing needs `full`.

## Send it

```bash
curl https://api.nimbus.example/v1/projects \
  -H "Authorization: Bearer nbp_2f8a1example"
```

Missing or bad credentials return `401 unauthenticated`; a valid token without the right scope or role returns `403 forbidden` — the split matters when debugging: 401 means fix the token, 403 means fix its scope.

## Expiry and revocation

Expired tokens start returning 401 at the expiry instant. Revoke any token immediately from **Account → Tokens**; revocation is instant and permanent. Set expiries on automation tokens (90 days is a sane default) so forgotten credentials die on their own.

## Tokens vs the CLI

The CLI uses the same tokens under the hood: `NIMBUS_TOKEN=nbp_... nimbus deploy` is the standard CI pattern (see [Install the CLI](/guides/getting-started/install-cli)). Anything you script against the raw API, the CLI's `--json` mode may already provide — check [the CLI reference](/reference/cli/overview) first.
