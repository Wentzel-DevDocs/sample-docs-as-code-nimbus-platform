---
title: Install the CLI
description: Install the nimbus CLI with npm or the install script, authenticate with nimbus login, and set up token auth for CI.
order: 1
---

# Install the CLI

The `nimbus` CLI is how you deploy, tail logs, and manage secrets from a terminal. It runs on macOS, Linux, and Windows (via WSL).

## Install

With npm (requires Node.js 18+):

```bash
npm install -g @nimbus-platform/cli
```

Or with the install script, which drops a standalone binary into `~/.nimbus/bin` and adds it to your `PATH`:

```bash
curl -fsSL https://get.nimbus.example | sh
```

Verify:

```bash
nimbus version
# nimbus 2.9.1 (darwin-arm64)
```

## Log in

```bash
nimbus login
```

This opens a browser to approve the device, then stores a credential in your OS keychain. Confirm who you are with:

```bash
nimbus whoami
# signed in as dev@example.com (org: acme-labs)
```

## Authenticating in CI

CI machines cannot run a browser flow. Create a personal access token with the `deploy` scope (see [Identity and access](/concepts/identity-and-access)) and export it:

```bash
export NIMBUS_TOKEN=nbp_2f8a1example
nimbus deploy --env production
```

When `NIMBUS_TOKEN` is set, the CLI uses it and skips the keychain entirely.

## Upgrading

- npm: `npm install -g @nimbus-platform/cli@latest`
- Install script: `nimbus upgrade` replaces the binary in place.

Next: [set up a project](/guides/getting-started/project-setup).
