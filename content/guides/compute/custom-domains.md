---
title: Custom domains
description: Attach your own domain to a service — DNS records, automatic TLS issuance and renewal, apex domains, and wildcard subdomains.
order: 3
---

# Custom domains

Every web and static service gets a platform URL; custom domains put your own name in front, with TLS handled for you.

## Attach a domain

Declare it on the service and deploy:

```yaml
services:
  - name: web
    type: web
    domains:
      - www.example.com
```

Or use the CLI without a deploy: `nimbus domains add www.example.com --service web`.

## Point DNS

Create a CNAME to the edge:

```text
www.example.com.  CNAME  edge.nimbusapp.example.
```

For an **apex** domain (`example.com`), CNAMEs are not allowed by DNS itself; use your DNS provider's ALIAS/ANAME record to the same target, or an A record to the anycast IPs listed under **Project → Networking** in the dashboard.

## TLS certificates

Once DNS resolves to the edge, Nimbus issues a certificate automatically — usually within a minute, occasionally up to an hour if DNS propagation is slow — and renews it forever after. Check progress with:

```bash
nimbus domains list
# www.example.com   web   dns:ok   tls:issued
```

The states are `dns:pending` → `dns:ok` and `tls:pending` → `tls:issued`. A domain stuck in `dns:pending` almost always means the CNAME has not propagated or points elsewhere; walk through [Troubleshooting → Networking](/troubleshooting/networking).

## Wildcards

`domains: ["*.customers.example.com"]` routes every subdomain to the service, and the request's `Host` header tells your app which tenant was addressed. Wildcard TLS requires a one-time DNS TXT challenge, which `nimbus domains add` prints for you.

## Redirects

The edge redirects HTTP to HTTPS automatically. To canonicalize (say, apex → `www`), do it in your app or with your DNS provider's redirect feature — Nimbus does not rewrite hostnames.
