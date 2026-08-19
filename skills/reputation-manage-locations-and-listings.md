---
name: Manage locations and listings
description: Create, search, deduplicate and audit the business locations that everything else in the Reputation platform hangs off.
api: openapi/reputation-locations-api-openapi.yml
operations: [get_locations, post_locations, get_locations_search, post_locations_search, get_locations_faceted_search, get_locations_locationID, delete_locations_locationID, get_locations_duplicate, put_locations_locationID_opt_in, put_locations_locationID_opt_out, post_locations_locationID_addPage, get_location_attribute_definitions, get_categories, get_listing_audits]
generated: '2026-08-13'
method: generated
source: openapi/reputation-locations-api-openapi.yml + openapi/reputation-categories-api-openapi.yml + openapi/reputation-listing-audits-api-openapi.yml + https://apidocs.reputation.com/
---

# Manage locations and listings

`locationID` is the pivot key for the whole platform — reviews, metrics, surveys, tickets and
listing audits are all scoped by it. Get locations right before anything else.

## Before you start

`X-API-KEY` on every request, plus `X-TENANT-ID` on an Agency key. A missing tenant id surfaces as
**404**, not 401.

## Steps

1. **Resolve the category taxonomy first** — `get_categories` (`GET /v3/categories`), paged with
   `offset`/`limit`. Then `get_location_attribute_definitions`
   (`GET /v3/location-attribute-definitions?categoryIds=…`) to learn which attributes are valid for
   the categories you chose. Attributes are category-dependent; sending an attribute a category does
   not define fails validation with **403**.
2. **Look before you create** — `get_locations` (`GET /v3/locations`) to page the whole set,
   `get_locations_search` / `post_locations_search` (`/v3/locations-search`) to query, or
   `get_locations_faceted_search` (`GET /v3/locations-faceted-search?groupBy=…`) to roll up by a
   facet.
3. **Create or update** — `post_locations` (`POST /v3/locations`).
4. **Fetch one** — `get_locations_locationID` (`GET /v3/locations/{locationID}`).
5. **Attach a source page** — `post_locations_locationID_addPage`
   (`POST /v3/locations/{locationID}/addPage`) to bind a third-party listing page to the location.
6. **Sweep duplicates** — `get_locations_duplicate` (`GET /v3/locations-duplicate`) and resolve
   what it returns before it pollutes metrics.
7. **Control audit participation** — `put_locations_locationID_opt_in`
   (`PUT /v3/locations/{locationID}/opt-in`) and `put_locations_locationID_opt_out`
   (`PUT /v3/locations/{locationID}/opt-out`).
8. **Read listing accuracy** — `get_listing_audits` (`GET /v3/listing-audits`) with
   `locationIDs`, `sourceIDs` and a `from`/`to`/`range` window.

## Destructive operation

`delete_locations_locationID` (`DELETE /v3/locations/{locationID}`) removes a location. There is no
undo endpoint in the published contract and no idempotency contract. Require explicit human
confirmation and log the `locationID` before calling it.

## Error handling

**403** on a write is usually validation, not permission — read `errors[].error.field` and
`errors[].error.message`. **404** covers a missing location id or a missing source id as well as a
genuinely absent record.
