---
generated: '2026-08-12'
method: generated
name: Build an audience and launch a campaign against it
description: Resolve a Blueshift segment or custom user list, attach it to a campaign, then run the campaign through its schedule/launch/pause/archive lifecycle via the API.
api: openapi/blueshift-openapi.yml
operations: [getV1SegmentsList, getV1SegmentsBySegmentUuidMatchingUsersJson, postV1CustomUserListsCreate, putV1CustomUserListsBulkAddUsersToListByListId, postV1CampaignsByCampaignType, patchV1CampaignsByCampaignUuidUpdateSchedule, patchV1CampaignsByCampaignUuidLaunch, patchV1CampaignsByCampaignUuidPause, putV1CampaignsByCampaignUuidArchive]
source: >-
  Grounded in openapi/blueshift-openapi.yml (operationIds assigned by
  overlays/blueshift-openapi-overlay.yaml). Batch caps and lifecycle constraints
  from conventions/blueshift-conventions.yml and
  errors/blueshift-problem-types.yml.
---

# Build an audience and launch a campaign against it

Two ways to define who receives a message, and they behave differently.

- A **segment** is criteria-driven — Blueshift evaluates it against profile attributes and events. Read-only over the API.
- A **custom user list** is an explicit membership list you maintain. Fully writable over the API.

## Auth

User API key as the HTTP Basic username, empty password. Region-matched base URL.

## Steps — segment path

1. **List segments** — `getV1SegmentsList` (`GET /api/v1/segments/list`).
2. **Check the audience size before you send** — `getV1SegmentsBySegmentUuidMatchingUsersJson` (`GET /api/v1/segments/{segment_uuid}/matching_users.json`) returns the count of users in the segment. Do this before launching anything large; campaign-level rate limiting only kicks in above a configured segment-size threshold.

Note: there is **no per-segment detail read and no segment create/update** in the public REST API. Segment authoring is console-only (or, in beta, via the MCP server's `create_segment_v2` / `update_segment_v2` tools).

## Steps — custom user list path

1. **Create the list** — `postV1CustomUserListsCreate` (`POST /api/v1/custom_user_lists/create`).
2. **Populate it** — `putV1CustomUserListsBulkAddUsersToListByListId` (`PUT /api/v1/custom_user_lists/bulk_add_users_to_list/{list_id}`). **25 users per call, or 500 with `async=true`.** Use `putV1CustomUserListsOverwriteListByListId` to replace membership wholesale rather than diffing.
3. **Read it back** — `getV1CustomUserListsIdByCustomUserListId` (`GET /api/v1/custom_user_lists/id/{custom_user_list_id}`).

## Steps — campaign lifecycle

1. **Create** — `postV1CampaignsByCampaignType` (`POST /api/v1/campaigns/{campaign_type}`) with the campaign attributes and the audience reference.
2. **Schedule** — `patchV1CampaignsByCampaignUuidUpdateSchedule` (`PATCH /api/v1/campaigns/{campaign_uuid}/update_schedule`).
3. **Launch** — `patchV1CampaignsByCampaignUuidLaunch` (`PATCH /api/v1/campaigns/{campaign_uuid}/launch`). Archived and completed campaigns cannot be launched; the API returns the same validation errors the campaign UI shows.
4. **Pause** — `patchV1CampaignsByCampaignUuidPause` (`PATCH /api/v1/campaigns/{campaign_uuid}/pause`). Only applies to running or scheduled campaigns.
5. **Archive** — `putV1CampaignsByCampaignUuidArchive`, or `putV1CampaignsBulkArchive` for a batch of UUIDs.

## Errors

- `422` is the lifecycle error. "Campaign cannot be paused in its current status", "Invalid schedule values". Read the message — the state machine is enforced server-side.
- `404` on a campaign UUID may be transient; retry once with backoff.

## Retry safety

**None of these operations accept an idempotency key.** `transaction_uuid` applies only to the campaign *execution* endpoints, not to campaign creation, scheduling or list mutation. A retried `postV1CampaignsByCampaignType` creates a second campaign. Confirm before retrying any write in this flow.

## Rate limiting to be aware of at launch

Campaign-level messaging rate limits override channel-level limits, but the **external-fetch limit overrides the campaign limit**. Blueshift's own example: an external-fetch limit of 5,000/min under a campaign limit of 10,000/min yields an effective 5,000/min. If the campaign uses an external fetch template, that is the ceiling. See `rate-limits/blueshift-rate-limits.yml`.
