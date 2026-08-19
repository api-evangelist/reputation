---
name: Monitor and respond to reviews
description: Pull new reviews across a brand's locations, read the responses already posted, and publish a brand-approved reply through the Reputation API.
api: openapi/reputation-reviews-api-openapi.yml
operations: [get_reviews3, get_reviews3_reviewID_responses, post_reviews3_reviewID_respond]
generated: '2026-08-13'
method: generated
source: openapi/reputation-reviews-api-openapi.yml + https://apidocs.reputation.com/
---

# Monitor and respond to reviews

Use the `reviews3` generation. The un-suffixed `/v3/reviews` operations (`get_reviews`,
`get_reviews_reviewID_responses`, `post_reviews_reviewID_respond`) still resolve but are the older
generation — prefer `reviews3` for new work.

## Before you start

- Send `X-API-KEY: <key>` on every request. Keys are issued by a Reputation customer success
  manager; there is no self-serve endpoint that mints one.
- If the key is an **Agency** key, also send `X-TENANT-ID: <tenantID>`. Omitting it returns **404**,
  not 400 — the docs list "Missing Tenant Id" under 404.
- Base host is `https://api.reputation.com` for US accounts and `https://api-eu.reputation.com`
  for EU accounts. They are separate data regions; a key is valid in one, not both.
- Set `Content-Type: application/json` on the POST.

## Steps

1. **List reviews** — `get_reviews3` (`GET /v3/reviews3`). Page with `offset` and `limit`; the
   default limit is 20. Read the `pagination` object from the response and follow `next` until it
   is absent. On newer endpoints `offset` may come back as a Base64 cursor string rather than an
   integer — pass whatever the server returned, do not re-derive it.
2. **Read existing responses** — `get_reviews3_reviewID_responses`
   (`GET /v3/reviews3/{reviewID}/responses`). Do this before replying. There is no idempotency key
   on the respond operation, so this read is the only way to avoid posting a duplicate reply.
3. **Post the reply** — `post_reviews3_reviewID_respond`
   (`POST /v3/reviews3/{reviewID}/respond`) with the response body as JSON.

## Error handling

- **403** has two meanings: the key is not permitted the operation, *or* the request body failed
  validation. Inspect `errors[].error.field` to tell them apart. A validation 403 is not retryable
  unmodified.
- **404** means the review, the location, the source or the tenant id is missing.
- **429** means the rate limit was exceeded. Reputation publishes no numeric limit and returns no
  `Retry-After` or `RateLimit-*` header, so back off exponentially and blindly.
- **500 / 504** — retry with backoff, then check `https://status.reputation.com` (the `API`
  component is tracked separately).

## Do not

- Do not retry `post_reviews3_reviewID_respond` on a timeout without first re-reading
  `get_reviews3_reviewID_responses`. There is no idempotency contract; a blind retry can double-post
  a public reply.
