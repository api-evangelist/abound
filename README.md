# Abound

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

Abound was a US tax-compliance API for platforms, marketplaces and fintechs serving the 1099
economy. Its v4 REST API covered the whole information-return lifecycle: collecting Form W-9,
W-8BEN and W-8BEN-E from payees, verifying their TIN against IRS records in real time, then
generating, filing, correcting, voiding and physically mailing Form 1099-NEC, 1099-MISC, 1099-K and
1099-INT to federal and state tax authorities. It also shipped drop-in UI components for payee
onboarding and a 44-event HMAC-signed webhook surface.

Backed by: 500-global — https://withabound.com

## Status: retired

Abound was acquired (announced November 2024) and the service has been shut down. Probed
2026-07-19:

- `withabound.com` has **no delegated nameservers** — the apex, `www`, `docs`, `status`,
  `production-api` and `sandbox-api` hosts all return NXDOMAIN.
- `github.com/withabound` returns **404**; the `abound-node` repository is gone.
- The domain is still registered (GoDaddy Corporate Domains, expires 2026-10-30) and locked with
  `serverUpdateProhibited` / `serverDeleteProhibited`.

The one surviving first-party artifact is the official npm package
[`@withabound/node-sdk`](https://www.npmjs.com/package/@withabound/node-sdk) (v6.0.68, last
published 2025-06-23). It is Fern-generated and ships the **complete first-party API definition**
under `.mock/definition/`. Every spec and artifact in this repo was recovered from it — nothing
here was reconstructed by guesswork.

## What is in this repo

| Path | What it is |
|---|---|
| `openapi/abound-v4-openapi.yml` | OpenAPI 3.1 — 55 operations, 39 paths, 149 schemas, 44 webhooks |
| `asyncapi/abound-webhooks-asyncapi.yml` | AsyncAPI 3.0 for the 44 HMAC-signed webhook events |
| `overlays/abound-v4-overlay.yaml` | API Evangelist annotations over the OpenAPI |
| `authentication/`, `conventions/`, `errors/` | Bearer auth, idempotency/pagination/ids, error envelopes |
| `data-model/` | Entity graph plus the 1099, TIN-verification and mailing state machines |
| `lifecycle/` | Versioning, the shutdown evidence, and the acquisition record |
| `sandbox/`, `components/`, `conformance/` | Historical sandbox, drop-in UI components, standards conformance |
| `packages/` | The surviving official SDK |
| `skills/` | Four Agent Skills grounded in real operationIds |
| `llms/`, `mcp/`, `agentic-access/`, `security/`, `well-known/` | Agent-facing and security artifacts |

Because the API is retired, none of the recorded endpoints are callable. The value here is as a
preserved reference model for information-return and tax-compliance API design.
