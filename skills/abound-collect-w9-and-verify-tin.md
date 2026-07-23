---
name: Collect a W-9 and verify the payee TIN
description: Onboard a US payee on the Abound API - create the user, collect a signed Form W-9, and confirm the IRS TIN match before any money moves.
api: openapi/abound-v4-openapi.yml
operations:
  - usersCreate
  - formW9Create
  - tinVerificationsRetrieve
  - tinVerificationsList
status: historical
---

# Collect a W-9 and verify the payee TIN

> **This API is retired.** Abound was acquired and the service has been shut down;
> `production-api.withabound.com` and `sandbox-api.withabound.com` no longer resolve. This skill
> documents the real operating procedure for the recovered v4 surface and is preserved as a
> reference model for information-return APIs. Do not expect live calls to succeed.

## Before you start

- Base URL: `https://production-api.withabound.com` (sandbox: `https://sandbox-api.withabound.com`).
- Auth: `Authorization: Bearer <appId>.<appSecret>` — the two credential halves joined by a period.
- Send an `Idempotency-Key` header on every POST. All three writes below accept one.
- Responses are bare JSON; errors are **not** RFC 9457. A `400` returns
  `{"errors":[{"field":…,"message":…}]}`; `401/404/409/500` return `{"message":…}`.

## Steps

1. **Create the user** — `usersCreate` (`POST /v4/users`).
   Pass your own `foreignId` in the body so you never have to keep a mapping table:
   `{"body":{"foreignId":"<your-internal-id>"}}`. A `409 Conflict` means this `foreignId` already
   exists — treat that as success and fetch the existing user with `usersList` filtered on it
   rather than retrying.

2. **Collect the W-9** — `formW9Create` (`POST /v4/documents/w-9`).
   Supply `payee` (name, full address, 9-digit `tin` with no hyphens, `tinType` of `INDIVIDUAL`
   for SSN/ITIN/ATIN or `BUSINESS` for EIN) and `formFields` (`taxClassification`,
   `isSubjectToBackupWithholding`, and an `electronicSignature` carrying `signature`,
   `printedName`, `signedAt` and `ipAddress`). Set `userId` to link the form to step 1.
   Creating the W-9 **automatically kicks off a TIN verification** when that name+TIN pair has not
   been seen before — do not create one by hand.

3. **Read the verification result.** The W-9 response embeds
   `payee.tinVerificationId` and `payee.tinVerificationStatus`. If the status is `PENDING`, poll
   `tinVerificationsRetrieve` (`GET /v4/tin-verifications/{tinVerificationId}`) or, better,
   subscribe to the `TIN_VERIFICATION_MATCH` / `TIN_VERIFICATION_MISMATCH` webhooks instead of
   polling.

4. **Gate on the result.**
   - `MATCH` — the payee is cleared; you may file information returns for them.
   - `MISMATCH` — do **not** file. Go back to the payee for corrected name/TIN details; filing on a
     mismatch is what drives a 1099 to `REJECTED`.
   - `PENDING` — wait; the IRS match is asynchronous.

## Reuse the fingerprint, not the TIN

Every verified payee gets a `payee.tinFingerprint` (`tinFingerprint_…`). On later calls you may
pass the fingerprint **instead of** the raw TIN. Prefer it: it keeps raw SSNs and EINs out of your
subsequent requests and logs. You can also filter document lists by `payeeTinFingerprint`.

## Related

- `conventions/abound-conventions.yml` — idempotency, pagination, identifier prefixes.
- `errors/abound-problem-types.yml` — the error envelopes.
- `asyncapi/abound-webhooks-asyncapi.yml` — the TIN verification and W-9 events.
