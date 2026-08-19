---
name: Request reviews and run surveys
description: Send review requests by email or SMS, generate a survey link, read the results back, and honour unsubscribes.
api: openapi/reputation-requests-api-openapi.yml
operations: [get_requests_templates, get_requests_unsubscribes, post_requests_send_email, post_requests_send_sms, post_requests_unsubscribe_email, post_requests_unsubscribe_phone, get_requests_metrics, post_surveys_link, get_survey_results, get_surveys3_templates]
generated: '2026-08-13'
method: generated
source: openapi/reputation-requests-api-openapi.yml + openapi/reputation-surveys-api-openapi.yml + https://apidocs.reputation.com/
---

# Request reviews and run surveys

Outbound flow. These operations send real email and SMS to real customers — treat every write as
non-reversible and check suppression first.

## Before you start

- `X-API-KEY` on every request; `X-TENANT-ID` as well on an Agency key.
- `Content-Type: application/json` on all POSTs.

## Steps

1. **Check suppression first** — `get_requests_unsubscribes`
   (`GET /v3/requests/unsubscribes`). Page it with `offset`/`limit` and exclude every listed
   address and phone number from your send list. Reputation publishes an anti-spam policy; sending
   to an unsubscribed recipient is a compliance failure, not just an API error.
2. **Pick a template** — `get_requests_templates` (`GET /v3/requests/templates`) for review
   requests, or `get_surveys3_templates` (`GET /v3/surveys3-templates`) for survey templates.
3. **Send** — `post_requests_send_email` (`POST /v3/requests/send-email`) or
   `post_requests_send_sms` (`POST /v3/requests/send-sms`).
4. **Or hand out a survey link instead** — `post_surveys_link` (`POST /v3/surveys-link`) returns a
   URL you can embed in your own channel. Use this when you own the delivery.
5. **Read results** — `get_survey_results` (`GET /v3/survey-results`), or
   `get_survey_results_surveyID` (`GET /v3/survey-results/{surveyID}`) with a
   `surveyTemplateID` query parameter for a single result.
6. **Measure** — `get_requests_metrics` (`GET /v3/requests/metrics`) for send/response performance
   over a date range. The response echoes the applied window in a `range` object (`range`, `from`,
   `to`) — read it back rather than assuming your requested window was honoured.
7. **Honour opt-outs inbound** — `post_requests_unsubscribe_email`
   (`POST /v3/requests/unsubscribe-email`) and `post_requests_unsubscribe_phone`
   (`POST /v3/requests/unsubscribe-phone`).

## Deprecated — do not use

`get_request_urls` (`GET /v3/request-urls`) is deprecated. Use `get_requests_request_urls`
(`GET /v3/requests/request-urls`). Likewise `get_surveys2_results`, `post_surveys2_results`,
`post_surveys2_results_create_url_from_encrypted`, `get_surveys3_results_surveyID` and
`post_surveys3_results` are deprecated in favour of the `/v3/survey-results` operations.
Reputation publishes no sunset dates for any of them, so migrate on your own schedule but do not
build new work on them.

## Error handling

There is **no idempotency key** on any send operation. A retried `post_requests_send_email` or
`post_requests_send_sms` after a timeout may send the message twice. Record your own send ledger
before the call and reconcile with `get_requests_metrics` instead of retrying blind. 429 carries no
`Retry-After`; back off exponentially.
