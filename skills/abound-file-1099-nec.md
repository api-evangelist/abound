---
name: Create, file and mail a Form 1099-NEC
description: Run the full Abound 1099-NEC lifecycle - create the document, file it with federal and state authorities, then mail the payee copy and track delivery.
api: openapi/abound-v4-openapi.yml
operations:
  - form1099NecCreate
  - form1099NecFile
  - form1099NecRetrieve
  - form1099NecMail
  - form1099NecDelete
  - mailingsRetrieve
status: historical
---

# Create, file and mail a Form 1099-NEC

> **This API is retired.** Abound was acquired and the hosts no longer resolve. This skill records
> the real v4 procedure recovered from the first-party API definition.

The same shape applies to `form1099Misc*`, `form1099K*` and `form1099Int*` — only the `formFields`
differ.

## Before you start

- Auth: `Authorization: Bearer <appId>.<appSecret>`.
- Put an `Idempotency-Key` on every POST below. Filing is not something you want to do twice.
- **All monetary values are in cents.**
- The payee's TIN should already be `MATCH` — see the *Collect a W-9 and verify the payee TIN*
  skill. Creating the 1099 will start a verification itself if the name+TIN pair is new.

## Steps

1. **Create the document** — `form1099NecCreate` (`POST /v4/documents/1099-nec`).
   Body needs `filingYear`, `payer` (name, `tin`, address, `phoneNumber`), `payee` (name, `tin` or
   `tinFingerprint`, address) and `formFields`. For the NEC the field that matters is
   `nonemployeeCompensation` (cents). Add `stateTaxInfo[]` with `filingState` and `stateIncome`
   for state filing, or use `filingState: "N/A"` to skip it. Optionally set `userId`.
   The response comes back with `status: CREATED`, a `payerUrl` (unmasked TINs) and a `payeeUrl`
   (payee TIN masked) pointing at generated PDFs.

2. **Check before you file.** `CREATED` only means the data validated and the PDFs rendered.
   Confirm `payee.tinVerificationStatus` is `MATCH` — filing on a `MISMATCH` is the main cause of
   a later `REJECTED`.

3. **File it** — `form1099NecFile` (`POST /v4/documents/1099-nec/{documentId}/file`).
   No body required. Moves the document to `FILED` with the federal and state authorities.
   Watch for `TEN99_NEC_ACCEPTED` and `TEN99_NEC_REJECTED` webhooks rather than polling —
   acknowledgement from the tax authorities is asynchronous and can be slow.

4. **Mail the payee copy** — `form1099NecMail`
   (`POST /v4/documents/1099-nec/{documentId}/mail`). Body carries `to` and `from` addresses. You
   get back a `Mailing` (`mailingId_…`) with `status: CREATED` and a PDF `url`. Track it with
   `mailingsRetrieve`, or subscribe to `MAILING_IN_TRANSIT`, `MAILING_DELIVERED` and
   `MAILING_RETURNED_TO_SENDER`.

## The rules that bite

- **Delete only works before an action.** `form1099NecDelete` is rejected once `/file`, `/correct`
  or `/void` has run against the document. Before that it is the clean way to discard a mistake.
- **Correct and void only work after `FILED`.** You cannot correct a `CREATED` document — delete
  and recreate it instead.
- `federalIncomeTaxWithheld` and `stateTaxWithheld` are validated as `min: 0, max: 0` on the NEC:
  Abound did not support withholding reporting on this form.
- `accountNumber` is capped at 20 characters and is auto-generated if you omit it.

## Status values

`CREATED` → validated, PDFs generated · `FILED` → TIN verified and filed · `ACCEPTED` →
acknowledged by the authorities · `REJECTED` → TIN mismatch or rejected by an authority.

## Related

- `skills/abound-correct-or-void-1099.md` — fixing a filed return.
- `data-model/abound-data-model.yml` — the document/mailing/correction graph.
