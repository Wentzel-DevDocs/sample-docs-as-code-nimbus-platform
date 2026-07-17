---
title: Architecture
description: How Nimbus is put together - a regional control plane, cells that run workloads, builders that produce images, and an edge that routes traffic.
nav_order: 1
---
You do not need this page to use Nimbus, but it makes everything else easier to reason about - especially [scaling](/guides/compute/scaling) and [troubleshooting](/troubleshooting/deploy-failures).

## The pieces

- **Control plane** - the API, scheduler, and release manager. It holds desired state: which services exist, which release each should run, how many instances. The [CLI](/reference/cli/overview) and [REST API](/reference/api/overview) talk only to the control plane.
- **Builders** - isolated build machines that turn your source into an OCI image, using your Dockerfile or a detected buildpack. Build logs stream back through the control plane during `nimbus deploy`.
- **Cells** - the data plane: fleets of machines in each [zone](/concepts/regions-and-zones) that actually run instances. Cells keep serving even if the control plane is briefly unreachable; you just cannot change desired state during that window.
- **Edge** - the routing layer that terminates TLS, applies [load balancing](/guides/networking/load-balancing), and forwards requests to healthy instances across zones.

## What a deploy actually does

1. `nimbus deploy` uploads your source and a builder produces an immutable image.
2. The control plane creates a **release** - image plus config, e.g. `r-42`. Releases never change; that is what makes [rollback](/guides/compute/services#releases-and-rollback) instant.
3. The scheduler starts new instances on cells, waits for [health checks](/guides/compute/services#health-checks) to pass, shifts edge traffic to them, then drains the old instances.

## Failure isolation

Instances of a service are spread across zones by default, so a single-zone failure degrades capacity instead of availability. The blast radius of a bad deploy is bounded too: if new instances never pass health checks, traffic never moves and the release fails - see [deploy failures](/troubleshooting/deploy-failures).
