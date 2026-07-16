---
title: Platform versioning
description: How Nimbus versions runtime defaults, CLI releases, and documentation publishes.
order: 1
status: stable
audience: all
category: concept
product: Nimbus Platform
owners: docs-writers
last_reviewed: 2026-07-16
---

# Platform versioning

| Surface | Version signal |
| --- | --- |
| Docs site publish | Git SHA (`VERSION` in CI) |
| CLI | `nimbus --version` semver |
| Runtime images | Monthly channel tags |
| These pages | [2024 platform](/versions/2024-platform) vs current |

Breaking platform changes appear in [Release notes](/release-notes) and, when paths move, in `config/redirects.json`.
