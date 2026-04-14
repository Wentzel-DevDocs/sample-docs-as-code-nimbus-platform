---
title: Projects, services, and deployments
description: REST endpoints for listing projects and services, reading service details, creating deployments, and rolling back.
order: 3
status: stable
audience: developers
category: reference
product: Nimbus Platform
owners: docs-writers
last_reviewed: 2026-04-14
---

# Projects, services, and deployments

These endpoints cover the compute side of the API: enumerating what exists, reading its state, and shipping new versions. Conventions (auth, pagination, errors) are in the [API overview](/reference/api/overview).

## Projects

`GET /v1/projects` — paginated list.

`GET /v1/projects/{id}` —

```json
{ "id": "prj_44aa", "name": "my-app", "region": "iad", "created": 1765000000 }
```

## Services

`GET /v1/projects/{id}/services` — every service with its live state:

```json
{
  "data": [
    { "id": "svc_a1b2", "name": "api", "type": "web",
      "status": "live", "instances": 3, "url": "https://api-my-app.nimbusapp.example",
      "live_deployment": "dep_8c31f2" }
  ],
  "has_more": false
}
```

`GET /v1/services/{id}` returns the same shape for one service, plus its `scale`, `size`, and `domains` as currently configured.

Service metrics live at `GET /v1/services/{id}/metrics` — parameters and series are documented in [Metrics](/guides/observability/metrics).

## Deployments

`GET /v1/services/{id}/deployments` — history, newest first. Each record:

```json
{ "id": "dep_8c31f2", "status": "live", "version": "9f31c2d", "message": "fix checkout",
  "created": 1768646400, "finished": 1768646522 }
```

`status` is one of `queued`, `building`, `deploying`, `live`, `failed`, `superseded`.

`POST /v1/projects/{id}/deployments` creates a deployment from the project's connected git source:

```bash
curl -X POST https://api.nimbus.example/v1/projects/prj_44aa/deployments \
  -H "Authorization: Bearer nbp_2f8a1example" \
  -d '{ "ref": "9f31c2d", "environment": "production", "services": ["api"], "message": "hotfix" }'
```

Returns `202` with the new `dep_...` record; poll `GET /v1/deployments/{id}` for progress. Requires the `deploy` [token scope](/reference/api/authentication) and a connected repository — source upload deploys are CLI-only.

`POST /v1/services/{id}/rollback` with `{ "deployment": "dep_5a09e1" }` re-points to a previous deploy, exactly like `nimbus rollback`.
