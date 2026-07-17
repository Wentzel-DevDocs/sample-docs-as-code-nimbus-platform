---
title: Deploy failures
description: Diagnose the two ways a deploy fails - build errors (exit 10) and health-check failures (exit 11) - and what to check for each.
nav_order: 1
---
A failed [`nimbus deploy`](/reference/cli/nimbus-deploy) never breaks production - traffic only moves after new instances pass health checks. The exit code tells you which half to debug.

## Exit 10: the build failed

The image never got made. Read the build log:

```bash
nimbus logs --build
```

The usual causes:

- **Dependency install fails** - lockfile out of sync (`npm ci` errors), private registry unreachable from the builder. Commit an updated lockfile; for private packages, pass credentials with `--build-arg`.
- **Wrong builder detected** - `builder: auto` guessed wrong (e.g. a repo with both `package.json` and a `Dockerfile`). Pin `builder:` explicitly in [`nimbus.yaml`](/reference/config/nimbus-yaml).
- **Out-of-memory during build** - large webpack/Vite builds. The build environment follows the service `size`; bump it or slim the build.
- **Validation errors** - unknown keys or bad values in `nimbus.yaml` fail before upload, with line numbers.

## Exit 11: health checks failed

The image built, instances started, but probes to `healthcheck.path` never went green - so the release was abandoned. Look at what the *new* instances said:

```bash
nimbus logs --service web --since 15m
```

Work down this list:

- **App listens on the wrong port** - it must bind the `ports:` entry; read `PORT` from the [environment](/reference/config/environment-variables) rather than hardcoding.
- **Binding 127.0.0.1** - bind `0.0.0.0` so the prober can reach it.
- **Health path missing or non-2xx** - `/healthz` must exist and return 2xx without auth.
- **Slow start** - migrations or warmup exceed the window; raise `healthcheck.grace` ([reference](/guides/compute/services)).
- **Crash on boot** - a missing variable or bad `${...}` [interpolation](/reference/config/environment-variables) kills the process; the log's first lines say which.

Reproduce probes locally with `curl -i localhost:8080/healthz` before redeploying.

## Exit 12: canceled

Someone pressed Ctrl+C or a newer deploy superseded this one ([last writer wins](/reference/cli/nimbus-deploy)). Nothing is wrong; the newest deploy's outcome is the one that matters.

## Stuck mid-deploy?

`nimbus services status web` shows instances by state. If instances flap between `starting` and `unhealthy`, it is an exit-11 pattern - the release will time out and abandon on its own. Still stuck after that, with a release neither live nor abandoned: check `status.nimbus.example`, then email `support@nimbus.example` with the release ID.
