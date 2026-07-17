---
title: Custom domain with TLS
description: Move shop.example.com onto Nimbus with zero downtime - verify ownership first, then flip the routing record.
nav_order: 4
---
This walkthrough puts `shop.example.com` in front of the `web` service with a valid certificate before any traffic moves - the order of DNS changes is what makes it zero-downtime. Concepts are in the [custom domains guide](/guides/networking/domains).

## 1. Register the domain

```bash
nimbus domains add shop.example.com --service web --env production
```

```text
Create these DNS records:
  TXT   _nimbus-challenge.shop   "nim-verify-8f3ka92j"
  CNAME shop                     cname.nimbusdns.example
Status: pending  (nimbus domains status shop.example.com to check)
```

## 2. Prove ownership first

At your DNS provider, create **only the TXT record** for now:

```text
_nimbus-challenge.shop.example.com  TXT  "nim-verify-8f3ka92j"
```

Nimbus polls; within a few minutes:

```bash
nimbus domains status shop.example.com
# shop.example.com  verified  certificate: issued
```

Because verification rides the TXT record alone, the certificate is issued **while your old host still serves all traffic**.

## 3. Flip the routing record

Now point traffic at Nimbus. Subdomain:

```text
shop  CNAME  cname.nimbusdns.example
```

Apex (`example.com` itself) - CNAMEs are not allowed at the apex, so use both A records:

```text
@  A  203.0.113.10
@  A  203.0.113.11
```

As resolver caches expire (your record's TTL), traffic shifts to Nimbus, which already holds a valid certificate - no browser ever sees an error. Lower the TTL to 300s a day in advance if you want the cutover quick.

## 4. Force HTTPS and confirm

`force_https: true` is the default, so HTTP already redirects. Verify end to end:

```bash
curl -sI http://shop.example.com | head -1     # HTTP/1.1 301 Moved Permanently
curl -sI https://shop.example.com | head -1    # HTTP/2 200
```

Renewal is [automatic](/guides/networking/domains) about 30 days before expiry. If `status` sticks at `pending`, the checklist in [networking issues](/troubleshooting/networking-issues) covers the three causes that account for nearly every case.
