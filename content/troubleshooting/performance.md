---
title: Performance
description: A working method for latency and throughput problems - find the bottleneck with logs and status, then fix the layer that is actually slow.
nav_order: 3
---
Slow is a symptom with four common causes on Nimbus: undersized instances, too few instances, a slow dependency, or the edge timing out on your app. Identify which before changing anything.

## First, look at the shape of the problem

```bash
nimbus services status web        # CPU, memory, instance count vs max
nimbus logs --service web --since 1h --level warn
```

- **CPU pinned near 100%, latency rises with traffic** -> scale out or up.
- **Memory climbing until restarts** -> a leak; the restart events appear in `status`.
- **CPU low but latency high** -> the time is elsewhere: database, KV, an external API.
- **504s at the client** -> responses exceeding the [idle timeout](/guides/networking/load-balancing); the request died at the edge, and your app may still be chewing on it.

## Scale problems

Sustained time at `max` instances means [autoscaling](/guides/compute/scaling) is capped:

```bash
nimbus services scale web --max 20
```

Latency spikes only at *cold start* points to [`min: 0`](/guides/compute/scaling) on a production service - give it a floor of 1-2. Single-threaded runtimes (Node) often profit more from **more small instances** than fewer large ones; `target_rps` scales on load directly when CPU is a poor proxy.

## Database bottlenecks

The classic signature: app CPU idle, latency high, [`nimbus pg status`](/guides/data/managed-postgres) shows connections at the plan limit or storage nearly full.

- Add a connection pooler in the app before upsizing the plan - each instance times `max` connections multiplies fast.
- Log slow queries in your framework; missing indexes dwarf plan size as a cause.
- Hot, small, repeated reads (sessions, flags) belong in [KV](/guides/data/key-value), not Postgres.

## Payload and egress costs

Large responses are slow to ship and [billed as egress](/concepts/billing). Serve big assets from [object storage](/guides/data/object-storage) with presigned URLs, and let the [static builder's cache headers](/tutorials/deploy-static-site) keep hashed assets out of your instances entirely.

## Measure between your services

Calls between your own workloads should use `.internal` names on the [private network](/guides/networking/private-networks) - public URLs add an edge round-trip and egress to every hop. `X-Request-Id` flows edge -> app -> [logs](/reference/cli/nimbus-logs), so one grep reconstructs a slow request across services.
