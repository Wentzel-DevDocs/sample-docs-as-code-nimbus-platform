---
title: Regions and zones
description: The six Nimbus regions, how zones isolate failures, and how to choose where a service runs.
nav_order: 2
---
A **region** is an independent geographic deployment of Nimbus. A **zone** is an isolated failure domain inside a region - separate power, network, and cooling. Every region has at least three zones, named `a`, `b`, and `c` (so `us-east-a`, `us-east-b`, ...).

## Available regions

| Region | Location |
| --- | --- |
| `us-east` | Northern Virginia, USA |
| `us-west` | Oregon, USA |
| `eu-west` | Ireland |
| `eu-central` | Frankfurt, Germany |
| `ap-south` | Mumbai, India |
| `ap-northeast` | Tokyo, Japan |

## How placement works

A service is pinned to **one region**, chosen by the `region` key in [nimbus.yaml](/reference/config/nimbus-yaml); its instances are spread across that region's zones automatically. Nimbus never moves a workload between regions on its own - moving is a config change plus a deploy.

Data products are regional too: [Postgres databases](/guides/data/managed-postgres) and [KV namespaces](/guides/data/key-value) live in one region, and [object storage buckets](/guides/data/object-storage) are stored in the region where they are created.

## Choosing a region

- **Latency**: pick the region closest to your users; cross-region round trips dominate response time for chatty apps.
- **Data residency**: workloads and their data stay in the chosen region, including backups - relevant for EU data in `eu-west` / `eu-central`. See the [FAQ](/troubleshooting/faq) for the formal statement.
- **Co-location**: put a service in the same region as the database it talks to; same-region, cross-zone latency is typically under 2 ms.

There is no default region - `nimbus init` asks, and the answer is recorded in `nimbus.yaml` so every environment of the service deploys to the same place.
