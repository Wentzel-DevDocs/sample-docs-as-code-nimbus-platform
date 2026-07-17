---
title: Custom domains
description: Point your own hostname at a service - DNS records, ownership verification, automatic TLS, and HTTPS redirects.
nav_order: 1
---
Every service already answers at `https://<service>-<project>.nimbusapp.example`. A custom domain puts your own hostname in front of it, with a certificate Nimbus issues and renews for you.

## Add the domain

```bash
nimbus domains add shop.example.com --service web
```

The command prints exactly which DNS records to create. There are two, and both matter:

| Purpose | Record | Value |
| --- | --- | --- |
| Route traffic (subdomain) | `CNAME shop` | `cname.nimbusdns.example` |
| Route traffic (apex, e.g. `example.com`) | `A @` | `203.0.113.10` and `203.0.113.11` |
| Prove ownership | `TXT _nimbus-challenge.shop` | token printed by the command |

Apex domains cannot carry a CNAME, hence the pair of A records; use both for redundancy.

## Verification and TLS

Nimbus polls DNS and moves the domain through `pending` to `verified`, then issues the certificate - typically a couple of minutes after the records propagate. Check progress with:

```bash
nimbus domains status shop.example.com
```

Certificates renew automatically about **30 days before expiry**; there is nothing to rotate by hand. If the domain sits in `pending`, work through [networking issues](/troubleshooting/networking-issues) - it is almost always a typo'd record or a proxying DNS provider intercepting the challenge.

## HTTPS redirects

HTTP is served only long enough to redirect. To make that explicit per service:

```yaml
force_https: true
```

With `force_https` (the default for new services), port 80 answers `301` to `https://` and everything else - including WebSocket upgrades - happens on 443 through the [load balancer](/guides/networking/load-balancing).

## Limits and removal

A project can hold **20 domains** ([quota table](/troubleshooting/faq)); one domain points at exactly one service, while a service can have many domains. `nimbus domains remove shop.example.com` detaches immediately. The end-to-end walkthrough, including a zero-downtime cutover from another host, is the [custom domain and TLS tutorial](/tutorials/custom-domain-tls).
