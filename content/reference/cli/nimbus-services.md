---
title: nimbus services
description: List, inspect, scale, and restart services from the command line.
nav_order: 3
---
```bash
nimbus services <subcommand> [flags]
```

Read and adjust running [services](/guides/compute/services). All subcommands respect `--project`/`--env` and `--json`.

## services list

```bash
nimbus services list
```

```text
NAME     STATUS    INSTANCES   SIZE     URL
web      healthy   3/3         small    https://web-storefront.nimbusapp.example
api      healthy   2/2         medium   internal-only
worker   degraded  1/2         small    internal-only
```

`degraded` means fewer healthy instances than desired - `status` on the service explains why.

## services status

```bash
nimbus services status web
```

Shows per-instance state (instance ID, zone, health, restarts, age), the active release, live [scaling](/guides/compute/scaling) settings, and recent scaling events. This is the first stop when something looks wrong; the second is [`nimbus logs`](/reference/cli/nimbus-logs).

## services scale

```bash
nimbus services scale web --min 4 --max 20
nimbus services scale web --size large
```

Applies immediately and persists server-side. Size changes roll instances one at a time; count changes add or drain instances. Remember these [override `nimbus.yaml`](/guides/compute/scaling) until the file catches up.

## services restart

```bash
nimbus services restart web
```

Rolling restart of all instances on the current release - same image, fresh processes. Useful after changing [environment variables](/reference/config/environment-variables), which take effect on restart. For a code change, use [`nimbus deploy`](/reference/cli/nimbus-deploy); for going back, use `nimbus releases rollback`.
