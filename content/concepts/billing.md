---
title: Billing
description: Usage-based pricing on compute-seconds, storage, and egress; the free allowance; budgets and alerts.
nav_order: 5
---
Nimbus bills on usage, monthly, per [organization](/concepts/projects-and-orgs). There are no per-seat charges - members are free.

## What is metered

| Meter | Unit | Notes |
| --- | --- | --- |
| Compute | compute-seconds | Each running instance accrues seconds weighted by its [size](/guides/compute/services#instance-sizes); `nano` is the 1x baseline |
| Storage | GB-month | Object storage, Postgres storage, KV, and [volumes](/reference/api/volumes) |
| Egress | GB | Traffic leaving Nimbus to the internet; traffic inside a [private network](/guides/networking/private-networks) is free |
| Builds | build-minutes | Time your deploys spend in the builder |

A [scaled-to-zero](/guides/compute/scaling#scale-to-zero) service accrues no compute-seconds while idle - that is the point of it.

## Free allowance

Every org gets a monthly allowance intended to keep hobby projects free: 250,000 compute-seconds (about one `nano` instance always-on), 5 GB-month storage, 10 GB egress, and 200 build-minutes. Usage beyond the allowance is billed at the published per-unit rates on your invoice.

## Watching and capping spend

```bash
nimbus billing usage              # month-to-date, broken down by project and meter
nimbus billing usage --project storefront --json
```

Owners can set a **budget** per org or per project. Alert thresholds email the owners at 50/80/100% of budget; a budget can optionally be **hard**, which blocks new deploys and scales services down to their configured minimum once exceeded (it never deletes data).

## Invoices

Invoices are issued on the 1st for the previous month, itemized by project and meter, and downloadable from the dashboard as PDF and CSV.
