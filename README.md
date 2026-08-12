# Blueshift

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

Blueshift is an AI-powered customer engagement and customer data platform (CDP) that unifies customer
profiles, product and content catalogs, and behavioural event streams, then activates them across email,
SMS, push, in-app messaging, mobile inbox, iOS Live Activities and on-site live content.

- Website: https://blueshift.com/
- Developer portal: https://developer.blueshift.com/
- API reference: https://developer.blueshift.com/reference/welcome
- Status: https://status.blueshift.com/
- Trust center: https://trust.blueshift.com/
- GitHub: https://github.com/blueshift-labs

## What this profile holds

| Artifact | Detail |
|---|---|
| `openapi/` | OpenAPI 3.0.0, 72 paths / 81 operations / 21 tags. Harvested verbatim from the per-operation OpenAPI documents Blueshift embeds in every `developer.blueshift.com/reference/*.md` page — there is no single downloadable spec. |
| `overlays/` | Overlay 1.0.0 adding the 81 `operationId`s Blueshift's published spec omits. |
| `mcp/` | Official OAuth-gated remote MCP server (public beta), plus Blueshift's own 131-tool catalogue and a crosswalk binding those tools to REST operations. |
| `well-known/` | RFC 8414 + RFC 9728 OAuth discovery documents, served on all four API/app hosts. No security.txt, no api-catalog, no agent card. |
| `postman/` | Blueshift's first-party public Postman collection — 83 requests across 22 folders. |
| `skills/` | Four packaged Agent Skills, every `operationId` verified against the spec. |
| `conventions/`, `errors/`, `rate-limits/`, `authentication/`, `scopes/` | Runtime semantics: idempotency via `transaction_uuid`, the error envelopes, the absent rate-limit headers, the two API-key classes and the eight OAuth scopes. |
| `conformance/`, `security/` | SOC 2 Type 2, ISO/IEC 27001:2022, HIPAA, GDPR, CCPA, EU-U.S. DPF — plus the standards the API does *not* implement. |
| `packages/`, `components/`, `sandbox/`, `plans/`, `lifecycle/`, `changelog/`, `data-model/`, `asyncapi/`, `llms/` | SDK currency, client surfaces, the sandbox account model, published pricing, lifecycle gaps, change signals, the entity graph and the webhook surface. |

No A2A agent card is recorded because Blueshift serves none — `/.well-known/agent-card.json` and the legacy
`/.well-known/agent.json` return 404 on every host probed.
