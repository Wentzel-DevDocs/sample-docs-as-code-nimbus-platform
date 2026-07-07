---
title: nimbus secrets
description: Full reference for nimbus secrets — set, list, and remove secrets per environment, bulk import from files, and how injection and restarts work.
order: 4
status: stable
audience: developers
category: reference
product: Nimbus Platform
owners: docs-writers
last_reviewed: 2026-07-07
---

# nimbus secrets

Secrets are encrypted values injected into service instances as environment variables at start time. They are write-only: settable and removable, never readable back (see [Identity and access](/concepts/identity-and-access)).

## Subcommands

```bash
nimbus secrets set KEY=VALUE [KEY2=VALUE2...]   # create or update
nimbus secrets list                              # names and last-changed times, never values
nimbus secrets rm KEY [KEY2...]                  # delete
```

All three take `--env <name>`; without it they act on `production`.

## Setting secrets

```bash
nimbus secrets set STRIPE_KEY=sk_live_abc DATABASE_POOL=8
nimbus secrets set SMTP_PASSWORD=- < password.txt     # read a value from stdin
nimbus secrets set --from-file .env.production        # bulk import KEY=VALUE lines
```

A change triggers a **rolling restart** of the environment's services so new values take effect — instances are replaced health-checked, exactly like a deploy, so there is no downtime but there is a minute of churn. Batch related changes into one `set`.

## Scoping to one service

By default secrets are project-wide within the environment. Restrict with `--service`:

```bash
nimbus secrets set MAILER_KEY=abc --service email-consumer
```

A service-scoped secret shadows a project-wide one of the same name for that service.

## Precedence

For the same variable name, the effective value is: service-scoped secret, then project secret, then `env:` in [nimbus.yaml](/reference/config/nimbus-yaml). Plain configuration belongs in `nimbus.yaml` where it is code-reviewed; anything sensitive belongs here. The full model is in [Environment variables](/reference/config/environment-variables).

## Names

Uppercase letters, digits, and underscores, not starting with a digit. Names prefixed `NIMBUS_` are reserved for [platform-injected variables](/reference/config/environment-variables).
