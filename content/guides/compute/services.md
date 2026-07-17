---
title: Services
description: Long-running HTTP workloads - instance sizes, health checks, zero-downtime releases, and rollback.
nav_order: 1
---
A service is a long-running workload that receives traffic: an API, a web app, a worker with an HTTP health endpoint. Services are declared in [`nimbus.yaml`](/reference/config/nimbus-yaml) and created on first [deploy](/reference/cli/nimbus-deploy).

## Instance sizes

| Size | vCPU | Memory |
| --- | --- | --- |
| `nano` | 0.25 | 256 MB |
| `small` | 0.5 | 512 MB |
| `medium` | 1 | 1 GB |
| `large` | 2 | 4 GB |
| `xlarge` | 4 | 8 GB |

Set `size` in `nimbus.yaml`, or override live with `nimbus services scale web --size large` (the change is applied as a rolling restart). Pricing is per compute-second - see [Billing](/concepts/billing).

## Health checks

Nimbus only sends traffic to instances that pass the health check:

```yaml
healthcheck:
  path: /healthz
  interval: 10s     # how often to probe
  timeout: 2s       # per-probe timeout
  grace: 30s        # startup grace period before probes count
```

An instance that fails three consecutive probes is restarted; during a deploy, failing probes [block the release](/troubleshooting/deploy-failures). Raise `grace` for apps with slow starts.

## Releases and rollback

Each deploy produces an immutable, numbered release (`r-1`, `r-2`, ...). Traffic switches to new instances only after they pass health checks, so releases are zero-downtime by construction.

```bash
nimbus releases list
nimbus releases rollback r-41    # instant - the old image is still present
```

A rollback is itself a new release (pointing at the old image), so history stays linear.

## Instances and zones

Instances are spread across [zones](/concepts/regions-and-zones) automatically. See what is running with:

```bash
nimbus services status web
```

Instance count is governed by [scaling](/guides/compute/scaling); persistent disks are covered under [volumes](/reference/api/volumes). To keep a service off the public internet entirely, set `internal_only: true` and reach it over the [private network](/guides/networking/private-networks).
