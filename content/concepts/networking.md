---
title: Networking
description: How traffic reaches a service — load balancing, private service-to-service DNS, egress IPs, and request limits.
order: 5
status: stable
audience: all
category: concept
product: Nimbus Platform
owners: docs-writers
last_reviewed: 2026-02-10
---

# Networking

This page describes the path a request takes into, between, and out of your services.

## Inbound: edge to service

HTTPS requests hit the nearest edge location, which terminates TLS and forwards to healthy instances of your web service in the project's [region](/concepts/regions-and-edge), balancing least-connections across instances. HTTP is redirected to HTTPS; HTTP/2 is used end to end when your service supports it.

Two headers are added on the way in:

| Header | Contents |
| --- | --- |
| `X-Forwarded-For` | Original client IP |
| `X-Request-Id` | Unique per request; echoed in [logs](/guides/observability/logs) |

## Request limits

| Limit | Value |
| --- | --- |
| Request body size | 100 MB |
| Request timeout | 300 seconds |
| WebSocket idle timeout | 5 minutes without frames |

Long-lived work beyond these limits belongs in a [worker](/concepts/services) fed by a queue, not in a request handler.

## Private networking between services

Services in the same project and environment reach each other over a private network at `<service>.internal` — for example, `http://api.internal:8080`. Internal traffic never leaves the region, is not TLS-terminated by the edge, and does not count toward request limits. Workers and cron jobs can call web services the same way.

## Outbound (egress)

Each project gets a stable set of egress IPs per region, listed under **Project → Networking** in the dashboard — use them to allowlist Nimbus at third-party APIs and databases. Egress to the internet is open by default.

## What is not configurable

There are no raw TCP/UDP inbound listeners (inbound is HTTP/S and WebSockets only), no VPC peering, and no per-service firewalls today. If a database requires IP allowlisting, use the project egress IPs. Connectivity problems are collected in [Troubleshooting → Networking](/troubleshooting/networking).
