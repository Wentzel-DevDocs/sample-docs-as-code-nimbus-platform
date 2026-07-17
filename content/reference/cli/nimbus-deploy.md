---
title: nimbus deploy
description: Flags, behavior, and exit codes for nimbus deploy - the build-release-verify command.
nav_order: 2
---
```bash
nimbus deploy [flags]
```

Reads [`nimbus.yaml`](/reference/config/nimbus-yaml), uploads the source, builds an image, starts instances, waits for [health checks](/guides/compute/services), and shifts traffic. If any step fails, the previous release keeps serving.

## Flags

| Flag | Default | Description |
| --- | --- | --- |
| `--project`, `--env` | from config | Target - see [precedence](/reference/cli/overview) |
| `--message <text>` | none | Annotation shown in `nimbus releases list` |
| `--no-wait` | off | Return after the build; do not wait for health checks |
| `--watch` | off | Keep streaming logs after release |
| `--build-arg KEY=VALUE` | none | Build-time variable (repeatable); not available at runtime |
| `--json` | off | Emit the release object on success |

## What a deploy does

1. **Upload** - the directory is packed, honoring `.nimbusignore`.
2. **Build** - the configured [builder](/reference/config/nimbus-yaml) produces an image, content-addressed by source digest. An unchanged commit skips straight to release, which is what makes [promotion](/guides/getting-started/environments) fast.
3. **Release** - new instances start alongside old ones.
4. **Verify** - health checks must pass within the grace period.
5. **Shift** - the [edge](/guides/networking/load-balancing) moves traffic; old instances drain and stop.

## Exit codes

| Code | Meaning |
| --- | --- |
| `0` | Release is live |
| `10` | Build failed - inspect with `nimbus logs --build` |
| `11` | Health checks failed - the release was abandoned, traffic never moved |
| `12` | Deploy canceled (Ctrl+C or a newer deploy superseded it) |

CI should treat any non-zero as failure; `10` vs `11` tells you whether to look at the [build or the app](/troubleshooting/deploy-failures).

## Concurrency

One deploy per service per environment runs at a time. Starting a second cancels the first (`12`) - last writer wins, and because traffic only shifts on success, the cancellation is safe.
