---
title: Networking issues
description: Fix domains stuck in pending, internal DNS that will not resolve, dropped WebSockets, and 413s on upload.
nav_order: 2
---
The four networking problems that generate most tickets, with the checks that resolve them.

## A domain sits in "pending"

`nimbus domains status shop.example.com` stays `pending` because verification cannot see your records. Three causes cover nearly every case:

1. **TXT record wrong or misplaced** - it must be exactly `_nimbus-challenge.<host>` with the printed token. Check what the world sees: `dig TXT _nimbus-challenge.shop.example.com +short`.
2. **A proxying DNS provider** - "proxy" or "CDN" modes intercept the hostname and hide your records. Set the record to DNS-only mode.
3. **Propagation** - TTLs and negative caching can delay visibility; if `dig` shows the right value, verification completes within minutes.

Also confirm the routing record matches the [guide](/guides/networking/domains): `CNAME` -> `cname.nimbusdns.example` for subdomains, the two `A` records (`203.0.113.10`, `203.0.113.11`) for the apex - an apex CNAME will be rejected or ignored by your provider.

## Internal DNS does not resolve

`curl http://api.storefront.internal:8080` fails from another service. Run the built-in diagnostic first:

```bash
nimbus network check api.storefront.internal --env production
```

It distinguishes the three failure modes:

- **Wrong environment** - `.internal` names only resolve [within one environment](/guides/networking/private-networks); a dev caller cannot see production's `api`.
- **Service down or renamed** - `nimbus services list` in that environment; a `stopped` or missing service has no DNS entry.
- **Wrong port** - internal traffic goes straight to the instance port (`ports:` in [`nimbus.yaml`](/reference/config/nimbus-yaml)); there is no 80/443 mapping on `.internal` names.

## WebSockets drop after exactly N seconds

That is the [idle timeout](/guides/networking/load-balancing) - 60s by default - closing a connection with no bytes moving. Send a ping inside the window, and raise the ceiling if needed:

```yaml
timeouts:
  idle: 300s   # maximum
```

If drops correlate with deploys instead, that is expected: instances drain on release, and clients should reconnect.

## Uploads fail with 413

The edge rejects request bodies over **100 MB**, and the limit is fixed. Upload large files directly to [object storage with a presigned URL](/guides/data/object-storage) and send your service the object key - faster for users and it takes your instances out of the data path.
