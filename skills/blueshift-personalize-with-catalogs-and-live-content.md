---
generated: '2026-08-12'
method: generated
name: Personalize a site or app with catalogs and live content
description: Load a Blueshift product/content catalog, then fetch personalized recommendations for a customer through a live content slot and render them in a website or mobile app.
api: openapi/blueshift-openapi.yml
operations: [postV1Catalogs, getV1Catalogs, putV1CatalogsByCatalogUuidJson, getV1CatalogsByCatalogUuidJson, getV1OnsiteSlotsJson, postLive]
source: >-
  Grounded in openapi/blueshift-openapi.yml (operationIds assigned by
  overlays/blueshift-openapi-overlay.yaml) and
  https://developer.blueshift.com/reference/live-content-slot. Component
  behaviour recorded in components/blueshift-components.yml.
---

# Personalize a site or app with catalogs and live content

Blueshift returns personalization as **data**, not as a rendered widget. You fetch JSON and render it yourself — there is no Blueshift web component or CDN bundle to embed.

## Auth — this flow spans both key classes

- Catalog operations use the **User API key**.
- `postLive` (the live-content fetch) uses the **Event API key**, because it is called from client-facing infrastructure.

That split is deliberate: the key you deploy to your edge for personalization cannot read or delete customer records.

## Steps — load the catalog

1. **Create a catalog** — `postV1Catalogs` (`POST /api/v1/catalogs`).
2. **List catalogs** — `getV1Catalogs` (`GET /api/v1/catalogs`) to find an existing `catalog_uuid`.
3. **Add items** — `putV1CatalogsByCatalogUuidJson` (`PUT /api/v1/catalogs/{catalog_uuid}.json`). **Hard cap of 100 items per call**, enforced with `413`. On partial failure you get `422` and the response identifies which items had invalid data — read it rather than retrying the whole batch.
4. **Verify** — `getV1CatalogsByCatalogUuidJson` (`GET /api/v1/catalogs/{catalog_uuid}.json`).

Note there is **no per-item read**. Items are only addressable through the parent catalog document, so plan your sync around whole-catalog reads.

## Steps — serve personalized content

1. **List the configured slots** — `getV1OnsiteSlotsJson` (`GET /api/v1/onsite_slots.json`). Slots are authored in the Blueshift app; the API only enumerates them.
2. **Fetch content for a customer** — `postLive` (`POST /live`) with the slot reference and a customer identifier. The response is a JSON payload of recommendations or content.
3. **Render it yourself.** Blueshift supplies no renderer for the web. Cache with care — the response is per-customer.

## Errors

- `413` — more than 100 catalog items in one call. Chunk it.
- `422` — some items carry invalid data; the body says which.
- `429` — back off exponentially; no `Retry-After` is returned.

See `errors/blueshift-problem-types.yml`.

## Retry safety

Catalog writes have **no idempotency key**. `putV1CatalogsByCatalogUuidJson` is a `PUT` that *adds* items, so a naive retry can duplicate them. Reconcile against `getV1CatalogsByCatalogUuidJson` rather than blind-retrying.

## Sandbox caveat that will bite you

In a sandbox account, recommendation engines, Photon V2 recipes and predictive features **degrade or do not work**, because a sandbox lacks the engagement history and catalog interaction data those models need. Blueshift's own advice is to test with random catalog selection or manually configured product sets. Do not conclude the recommendation integration is broken because a sandbox returned nothing. See `sandbox/blueshift-sandbox.yml`.
