---
title: Regions and the edge
description: The Nimbus regions, how to choose one, what runs at the edge, and how region migration works.
order: 4
status: stable
audience: all
category: concept
product: Nimbus Platform
owners: docs-writers
last_reviewed: 2026-03-10
---

# Regions and the edge

Every project has a **primary region** where its service instances, Postgres databases, and Store buckets live. In front of every region sits the **edge network**, which terminates TLS and serves static content close to users everywhere.

## Available regions

| Code | Location |
| --- | --- |
| `iad` | Ashburn, US East |
| `sfo` | San Francisco, US West |
| `fra` | Frankfurt, Europe |
| `sin` | Singapore, Asia Pacific |
| `syd` | Sydney, Oceania |

Set the region once in [nimbus.yaml](/reference/config/nimbus-yaml):

```yaml
name: my-app
region: fra
```

## Choosing a region

Put the region next to your data's users or its legal home. Latency between your web service and its [Postgres database](/guides/data/nimbus-postgres) matters more than latency to end users, because the edge already absorbs much of the user-facing distance: static assets, TLS handshakes, and cached responses are served from whichever edge location is closest to the visitor.

## What runs at the edge

- TLS termination and certificate serving for platform and [custom domains](/guides/compute/custom-domains).
- All `static` services, in every region, automatically.
- Response caching for web services when responses carry standard `Cache-Control` headers.

Dynamic requests are forwarded to the primary region over Nimbus's backbone.

## Moving a project between regions

Changing `region` in `nimbus.yaml` and deploying moves **compute only**. Resources do not move automatically: Postgres and Store data stay put until you migrate them (dump/restore for Postgres, `nimbus store sync` for buckets). Plan a maintenance window; cross-region reads in the interim are slower but functional. KV namespaces are globally replicated and need no migration — see [Key-value store](/guides/data/key-value-store).
