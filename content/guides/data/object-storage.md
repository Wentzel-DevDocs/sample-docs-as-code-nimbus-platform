---
title: Object storage (Nimbus Store)
description: Attach a Store bucket, read and write objects over the REST API, generate signed URLs for uploads and downloads, and serve public files.
order: 1
status: stable
audience: developers
category: how-to
product: Nimbus Platform
owners: docs-writers
last_reviewed: 2026-07-01
---

# Object storage (Nimbus Store)

Nimbus Store holds files — uploads, exports, images — in buckets attached to your project. Objects are replicated within the project's [region](/concepts/regions-and-edge) and served through the edge.

## Attach a bucket

```yaml
resources:
  - name: uploads
    type: store
```

Deploying provisions the bucket and injects two environment variables into every service: `NIMBUS_STORE_UPLOADS_URL` (the bucket's API base) and `NIMBUS_STORE_UPLOADS_KEY` (a bucket-scoped credential). Add `per_environment: true` to give [previews](/concepts/projects-and-environments) their own empty bucket.

## Read and write objects

The bucket speaks plain HTTP through the [storage API](/reference/api/storage):

```bash
# write
curl -X PUT "$NIMBUS_STORE_UPLOADS_URL/objects/reports/2026-07.pdf" \
  -H "Authorization: Bearer $NIMBUS_STORE_UPLOADS_KEY" \
  --data-binary @2026-07.pdf

# read
curl "$NIMBUS_STORE_UPLOADS_URL/objects/reports/2026-07.pdf" \
  -H "Authorization: Bearer $NIMBUS_STORE_UPLOADS_KEY" -o report.pdf
```

Keys are arbitrary UTF-8 paths; "folders" are just key prefixes. List with `GET /objects?prefix=reports/`, delete with `DELETE /objects/<key>`. Objects can be up to 5 TiB, uploaded in one request up to 5 GiB and multipart beyond that.

## Signed URLs

Never proxy large files through your web service, and never ship the bucket key to a browser. Mint short-lived signed URLs server-side instead:

```bash
curl -X POST "$NIMBUS_STORE_UPLOADS_URL/sign" \
  -H "Authorization: Bearer $NIMBUS_STORE_UPLOADS_KEY" \
  -d '{ "method": "PUT", "key": "avatars/u_31.png", "expires_in": 900 }'
# { "url": "https://uploads-my-app.store.nimbusapp.example/objects/avatars/u_31.png?sig=..." }
```

The browser then PUTs directly to the signed URL. The same endpoint signs `GET` for private downloads.

## Public buckets

`public_read: true` on the resource makes `GET`s anonymous and edge-cached — right for site assets and public images; keep user data private and signed.

The complete endpoint list is in the [storage API reference](/reference/api/storage).
