# Abound

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
