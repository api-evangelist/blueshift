---
generated: '2026-08-12'
method: generated
name: Ingest a customer profile and their behavioural events
description: Create or update a Blueshift customer profile, then send the behavioural events that drive segmentation, recommendations and event-triggered campaigns — single and bulk, with the correct key class and batch caps.
api: openapi/blueshift-openapi.yml
operations: [postV1Customers, postV1CustomersBulk, postV1Event, postV1Bulkevents, getV1EventDebug, getV1EventHistory]
source: >-
  Grounded in openapi/blueshift-openapi.yml (operationIds assigned by
  overlays/blueshift-openapi-overlay.yaml, since Blueshift's published spec
  declares none). Cross-cutting rules from conventions/blueshift-conventions.yml,
  errors/blueshift-problem-types.yml, authentication/blueshift-authentication.yml
  and rate-limits/blueshift-rate-limits.yml.
---

# Ingest a customer profile and their behavioural events

This is the foundation flow — nothing else in Blueshift works until profiles and events are landing.

## Pick the right base URL first

- US and rest of world: `https://api.getblueshift.com`
- EU: `https://api.eu.getblueshift.com`

The base URL **must** match the region the account is provisioned in, or every request fails. This is the most common first-integration error.

## Auth — and note the two key classes

Blueshift uses HTTP Basic with the API key as the **username** and an **empty password** (`-u <KEY>:`). There are two keys and they are not interchangeable:

- **Event API key** — use for `postV1Event` and `postV1Bulkevents`. Visible to any role, so it is the one that belongs in a server-side event collector.
- **User API key** — use for `postV1Customers` and `postV1CustomersBulk`. Admin-generated, and it can also delete customer PII, so scope its deployment tightly.

Using the wrong key returns `401`. See `authentication/blueshift-authentication.yml`.

## Steps

1. **Create or update the profile** — `postV1Customers` (`POST /api/v1/customers`) with the User API key. Supply at least one identifier: `email`, `customer_id` (your own id), or a Blueshift `uuid`. Custom attributes are free-form; see the data types guide for accepted types.
2. **Backfill in bulk** — `postV1CustomersBulk` (`POST /api/v1/customers/bulk`) for migrations. **Hard cap of 50 customers per call**, enforced with `413`. Blueshift's recommended throughput is 5 bulk calls per second (~250 users/sec).
3. **Send a single event** — `postV1Event` (`POST /api/v1/event`) with the Event API key. Include an identifier matching the profile and an event name. Reserved keys are prefixed `_bsft_`.
4. **Send events in bulk** — `postV1Bulkevents` (`POST /api/v1/bulkevents`). Recommended throughput is 5 bulk calls per second (~150 events/sec).
5. **Verify ingestion** — `getV1EventDebug` (`GET /api/v1/event/debug`) returns the latest event received per event type. `getV1EventHistory` (`GET /api/v1/event/history`) returns up to 50 recent successful events for a named event.

## Errors

- `400` — malformed or missing fields. Body is `{"errors": {"<field>": ["can't be blank"]}}`. Fix and resend; do not retry unchanged.
- `401` — wrong key class, or wrong region base URL.
- `413` — batch cap exceeded. Split the payload.
- `422` — semantically invalid; the response body identifies which records failed.
- `429` — slow down and back off exponentially (1s, 2s, 4s, 8s, jitter, ~5 attempts).

Full catalogue in `errors/blueshift-problem-types.yml`.

## Retry safety — read this before adding a retry wrapper

**Neither of these ingestion endpoints accepts an idempotency key.** Blueshift's `transaction_uuid` idempotency field only applies to `postV1CampaignsExecute` and `postV1CampaignsBulkExecute`. Retrying a customer or event write is **at-least-once**: a retried event will be counted twice and can re-fire an event-triggered campaign. Deduplicate on your side, or make the event carry a stable id you can reconcile against `getV1EventHistory`.

## Rate limits

There are **no rate-limit response headers** — no `X-RateLimit-*`, no `RateLimit-*`, no `Retry-After`. You cannot read your remaining budget. Stay at or under the published bulk guidance and treat `429` as the only signal. See `rate-limits/blueshift-rate-limits.yml`.

## Privacy operations

`postV1CustomersForget`, `postV1CustomersUnforget` and `postV1CustomersDelete` are the GDPR/CCPA affordances. They are permanent and sit behind the same User API key as everything else — gate them in your own layer.
