---
title: Load balancing
description: How the edge routes requests - protocols, sticky sessions, idle timeouts, and body-size limits.
nav_order: 3
---
Public traffic enters through Nimbus's edge, which terminates TLS and balances requests across your healthy instances. You do not configure a load balancer as a resource; you tune its behavior per service in [`nimbus.yaml`](/reference/config/nimbus-yaml).

## Protocols

The edge speaks **HTTP/1.1, HTTP/2, gRPC, and WebSockets** to clients. To your instances it forwards HTTP/1.1 (or HTTP/2 for gRPC backends) on your declared port, adding the usual headers: `X-Forwarded-For`, `X-Forwarded-Proto`, and `X-Request-Id` (echoed in [logs](/reference/cli/nimbus-logs), quote it in support tickets).

## Distribution and stickiness

Requests go to the least-busy healthy instance. Instances failing their [health check](/guides/compute/services) are removed from rotation until they recover.

If your app keeps per-user state in memory, enable sticky sessions:

```yaml
sticky: true
```

The edge then sets a `nimbus_affinity` cookie pinning each client to an instance while it lives. Stickiness is best-effort - deploys and [scale-in](/guides/compute/scaling) retire instances - so treat it as an optimization, not a correctness mechanism; durable state belongs in [Postgres](/guides/data/managed-postgres) or the [key-value store](/guides/data/key-value).

## Timeouts and limits

| Setting | Default | Maximum |
| --- | --- | --- |
| Idle timeout (`timeouts.idle`) | 60s | 300s |
| Request body size | 100 MB | 100 MB (fixed) |

```yaml
timeouts:
  idle: 120s
```

The idle timeout applies between bytes, so long-lived WebSockets survive as long as *something* flows - send a ping inside the window. Slow responses that exceed it surface to clients as `504`; uploads over 100 MB are rejected with `413` at the edge (put large objects in [object storage](/guides/data/object-storage) via presigned URLs instead). Both symptoms are unpacked in [performance troubleshooting](/troubleshooting/performance).
