---
title: Networking issues
description: Diagnosing custom domain and TLS problems, DNS propagation, internal service DNS failures, egress allowlists, WebSocket drops, and timeouts.
order: 3
status: stable
audience: all
category: troubleshooting
product: Nimbus Platform
owners: docs-writers
last_reviewed: 2026-07-07
---

# Networking issues

Symptoms and fixes for traffic that does not flow the way [Networking](/concepts/networking) says it should.

## Custom domain stuck at `dns:pending`

`nimbus domains list` shows the state machine from [Custom domains](/guides/compute/custom-domains). Stuck at `dns:pending` means the edge cannot see your record:

```bash
dig +short CNAME www.example.com
# expected: edge.nimbusapp.example.
```

No answer → the record was not created, or propagation (up to your zone's TTL) has not finished. A different answer → another record (often an old A) is shadowing the CNAME; remove it. Apex domains cannot use CNAME at all — use ALIAS/ANAME or the anycast A records from the dashboard.

## TLS stuck at `tls:pending`

Issuance starts only after `dns:ok`, and normally completes in about a minute. Persistent `tls:pending` with good DNS usually means a CAA record on the zone forbids issuance — add one permitting Nimbus's CA (shown in the dashboard's domain panel) or remove the restriction.

## `<service>.internal` does not resolve

Internal DNS works only between services in the **same project and environment**. Check the caller's `NIMBUS_ENV` — a preview cannot call production's `api.internal`. Also confirm the target port: internal calls go straight to the instance port (`http.port`), not 443, and are plain HTTP.

## Third-party API or database refuses connections

If the other side allowlists IPs, it must allowlist your project's egress IPs (dashboard → **Project → Networking**). Note the list is per [region](/concepts/regions-and-edge) — moving regions changes your egress IPs, which silently breaks allowlists.

## WebSockets drop after 5 minutes

That is the idle timeout from [Networking](/concepts/networking). Send a ping/pong heartbeat under the five-minute mark; active connections are not dropped.

## Requests time out at 300 seconds

The router's hard cap. Work that legitimately exceeds it belongs in a [background worker](/guides/compute/background-workers) with the client polling or receiving a webhook — raising the timeout is not available.

## Everything is slow from one geography

Static content is served at the edge, but dynamic requests travel to the project's region. If the slow geography is where your users are and your region is not, that is a [region choice](/concepts/regions-and-edge) problem, not an incident.
