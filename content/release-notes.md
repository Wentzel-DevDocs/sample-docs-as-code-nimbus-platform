---
title: Release notes
description: Platform-level Nimbus changes that affect operators — CLI flags, runtime defaults, and regional capacity.
order: 5
status: stable
audience: all
category: reference
product: Nimbus Platform
owners: docs-writers
last_reviewed: 2026-07-16
---

# Release notes

Sample release notes for an enterprise platform docs site. Product dates are fictional.

## 2026.7 — Preview environments per pull request

- Connected GitHub repos can create ephemeral environments per open PR.
- Previews sleep after 30 minutes of idle traffic; wake on the next request.
- See [Projects and environments](/concepts/projects-and-environments).

## 2026.5 — Postgres major version 16 default

- New `nimbus-postgres` resources default to PostgreSQL 16.
- Existing 15 instances are supported through the next two quarters.
- Backup retention defaults remain 7 days (configurable).

## 2026.3 — CLI `nimbus logs --since`

- `--since` accepts durations (`15m`, `2h`) or RFC3339 timestamps.
- JSON log lines gain `deploy_id` for correlation with [metrics](/guides/observability/metrics).

## How we version docs

Docs publish from `main` on every merge. The hosted site version id is the git SHA (or a CI-supplied `VERSION`). Breaking platform changes are called out here and in the relevant reference pages in the same PR.
