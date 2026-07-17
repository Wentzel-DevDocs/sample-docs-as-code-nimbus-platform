---
title: Private networks
description: Every environment gets a private network with internal DNS - service-to-service calls that never touch the internet.
nav_order: 2
---
Each [environment](/guides/getting-started/environments) has its own private network. Everything in the environment - services, jobs, [Postgres](/guides/data/managed-postgres) - is attached to it automatically; nothing in another environment or another project can reach it.

## Internal DNS

Every service gets an internal name:

```text
<service>.<project>.internal
```

So from the `web` service, the `api` service in the same environment's `storefront` project is simply:

```bash
curl http://api.storefront.internal:8080/v2/products
```

Internal DNS resolves only inside the environment. Which environment's `api` you reach is decided by where the *caller* runs - the name is identical in staging and production, which is what makes one `nimbus.yaml` portable across environments.

## Internal-only services

A service that should never be public - an internal API, an admin panel - opts out of the edge entirely:

```yaml
internal_only: true
```

It gets no `nimbusapp.example` URL, cannot have [custom domains](/guides/networking/domains), and is reachable only via its `.internal` name. Traffic to internal-only services skips the [load balancer](/guides/networking/load-balancing) and goes instance-to-instance.

## What travels the private network

- Service-to-service HTTP/gRPC via `.internal` names.
- Connections to attached data stores - the `DATABASE_URL` that `nimbus pg attach` injects points at a private address.
- [Job](/guides/compute/jobs) instances talking to services and data.

Traffic on the private network is encrypted in transit and does not count as [egress](/concepts/billing). Calling another service via its *public* URL, by contrast, exits and re-enters - slower, metered, and it defeats `internal_only` - so always prefer `.internal` names between your own workloads.

If an internal call fails, `nimbus network check` diagnoses the usual suspects - see [networking issues](/troubleshooting/networking-issues).
