---
generated: '2026-08-12'
method: generated
name: Trigger a transactional campaign without sending it twice
description: Fire an event-triggered or API-triggered Blueshift campaign for one customer or in batch, using the transaction_uuid idempotency field so a retry cannot deliver a duplicate message.
api: openapi/blueshift-openapi.yml
operations: [postV1CampaignsExecute, postV1CampaignsBulkExecute, getV2CampaignsJson, getV1CampaignsByCampaignUuidDetailJson]
source: >-
  Grounded in openapi/blueshift-openapi.yml (operationIds assigned by
  overlays/blueshift-openapi-overlay.yaml). Idempotency semantics quoted from
  https://developer.blueshift.com/reference/post_api-v1-campaigns-execute and
  captured in conventions/blueshift-conventions.yml.
---

# Trigger a transactional campaign without sending it twice

Order confirmations, password resets and system alerts. This is the one flow in the Blueshift API where duplicate delivery is unacceptable — and, not coincidentally, the only one that offers idempotency.

## Auth

User API key as the HTTP Basic username, empty password. Match the base URL to the account region (`https://api.getblueshift.com` or `https://api.eu.getblueshift.com`).

## Idempotency — a body field, not a header

Blueshift does **not** use the `Idempotency-Key` header. It uses a `transaction_uuid` field **in the request body**, and it must be a well-formed UUID.

If a request arrives with a `transaction_uuid` that has already been processed, Blueshift returns `200 OK` with a message noting the request was already processed, and **no second message is sent**.

The practical consequence: idempotency here cannot be added by a gateway, proxy or generic retry middleware. It has to be threaded through your application payload. Generate the UUID from something stable on your side — the order id, the reset-token id — not randomly per attempt, or the protection does nothing.

## Steps

1. **Find the campaign UUID** — `getV2CampaignsJson` (`GET /api/v2/campaigns.json`) filtered by name, status or type. Supports `page` and `per_page`. You can also read it out of the app URL: `app.getblueshift.com/dashboard#/app/campaigns/<CAMPAIGN_UUID>/details`.
2. **Trigger for one customer** — `postV1CampaignsExecute` (`POST /api/v1/campaigns/execute`) with `campaign_uuid`, a customer identifier (`email` or `customer_id`), your `transaction_uuid`, and any personalization data the template expects.
3. **Trigger in batch** — `postV1CampaignsBulkExecute` (`POST /api/v1/campaigns/bulk_execute`) for daily digests or batched transactional sends. Each entry carries its own `transaction_uuid`. Set `_bsft_high_priority: true` for time-sensitive campaigns — Blueshift processes high-priority FIFO ahead of standard FIFO. File attachments go in `email_attachments` as URLs.
4. **Confirm the result** — `getV1CampaignsByCampaignUuidDetailJson` (`GET /api/v1/campaigns/{campaign_uuid}/detail.json`) for the campaign report with stats.

## Watch for the sandbox

If the account is a sandbox (`account_mode = sandbox`), campaign execution is **blocked** and returns:

- `422 Unprocessable Entity`, error code `101`, message `<account> is a sandbox account`

This is the only numeric error code Blueshift publishes anywhere in its API. Test sends of templates still work in sandbox and do not consume quota. See `sandbox/blueshift-sandbox.yml`.

## Errors

- `404` — campaign not found. Blueshift also documents `404` as a possible *transient* conflict outcome, so retry with backoff once before concluding the campaign is gone.
- `422` — campaign is archived, completed, or otherwise not in a triggerable state.
- `429` — back off exponentially. No `Retry-After` is returned.
- `409` / `5xx` — retry with exponential backoff. Because you are sending `transaction_uuid`, these retries are safe.

See `errors/blueshift-problem-types.yml`.

## Tracking the outcome

The REST response tells you the trigger was accepted, not that a message was delivered. For delivery-side signal, configure the **campaign execution status webhook** (fires per qualifying trigger, `status: success|failed`) and the **campaign activity export** (delivered / opened / clicked / bounced / unsubscribed, batched every minute).

Both are configured in the Blueshift app under Account Settings > Campaign Activity Export — there is **no API to register a webhook**. Neither webhook is signed, so authenticate the receiver yourself (the activity export supports HTTP Basic). See `asyncapi/blueshift-webhooks.yml`.
