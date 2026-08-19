---
name: Read reputation metrics and triage tickets
description: Pull the reputation summary and page/post metrics for a date range, then open and route tickets against the feedback that needs action.
api: openapi/reputation-metrics-api-openapi.yml
operations: [get_summary, get_page_metrics, get_post_metrics, get_aggregate_page_metrics, get_aggregate_post_metrics, get_tickets, post_tickets, get_ticket_queues, get_ticket_stages, get_ticket_types, put_reports_reportID_export]
generated: '2026-08-13'
method: generated
source: openapi/reputation-metrics-api-openapi.yml + openapi/reputation-summary-api-openapi.yml + openapi/reputation-tickets-api-openapi.yml + openapi/reputation-reports-api-openapi.yml + https://apidocs.reputation.com/
---

# Read reputation metrics and triage tickets

## Before you start

`X-API-KEY` on every request, plus `X-TENANT-ID` on an Agency key. Metric and report queries are
the ones most likely to hit **504 Timeout** — narrow the date range or lower `limit` rather than
retrying the same query.

## Read the numbers

1. **Headline** — `get_summary` (`GET /v3/summary`).
2. **Per-page** — `get_page_metrics` (`GET /v3/page-metrics`) and `get_post_metrics`
   (`GET /v3/post-metrics`).
3. **Rolled up** — `get_aggregate_page_metrics` (`GET /v3/aggregate-page-metrics`) and
   `get_aggregate_post_metrics` (`GET /v3/aggregate-post-metrics`).
4. Every date-range endpoint echoes a `range` object (`range`, `from`, `to`) describing the window
   actually applied. Read it back and report against it — do not assume your requested window was
   honoured verbatim.

`get_metrics` (`GET /v3/metrics`) is **deprecated**; use the page/post metric operations instead.

## Export a report

`put_reports_reportID_export` (`PUT /v3/reports/{reportID}/export`). Note the EU region exposes this
as `PUT /v3/reports/{reportID}/run` instead — check which docs collection applies to your account
(`apidocs.reputation.com` for US, `apidocs-eu.reputation.com` for EU) before wiring it.

## Triage

1. **Load the vocabulary** — `get_ticket_queues` (`GET /v3/ticket-queues`), `get_ticket_stages`
   (`GET /v3/ticket-stages`), `get_ticket_types` (`GET /v3/ticket-types`). Queue, stage and type are
   enumerations defined by the tenant, not free text; resolve them before creating anything.
2. **List** — `get_tickets` (`GET /v3/tickets`), paged with `offset`/`limit`.
3. **Create** — `post_tickets` (`POST /v3/tickets`). Bind it to the originating review with
   `reviewID` so the review and the ticket stay linked in both directions.

## Error handling

No idempotency key exists on `post_tickets`. On a timeout, re-read `get_tickets` filtered by the
originating `reviewID` before retrying, or you will open duplicate tickets on the same review.
