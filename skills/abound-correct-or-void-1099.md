---
name: Correct or void a filed 1099
description: Amend a Form 1099 after it has been filed with the tax authorities - issue a correction or a void, and follow the document chain that links them.
api: openapi/abound-v4-openapi.yml
operations:
  - form1099NecCorrect
  - form1099NecVoid
  - form1099NecRetrieve
  - form1099NecList
status: historical
---

# Correct or void a filed 1099

> **This API is retired.** Recorded from the recovered Abound v4 definition.

Applies identically to `form1099Misc*`, `form1099K*` and `form1099Int*`.

## The precondition everyone trips on

**A 1099 can only be corrected or voided once it has reached `FILED`.** If the document is still
`CREATED`, there is nothing filed to amend — call `form1099NecDelete` and create a fresh one. Once
any of `/file`, `/correct` or `/void` has run, delete is permanently unavailable.

## Correcting

`form1099NecCorrect` — `POST /v4/documents/1099-nec/{documentId}/correct`, with an
`Idempotency-Key`.

Send the corrected `payee` and the corrected `formFields`. This does **not** mutate the original.
It files a **new** document and links the two. Abound automatically decides between a
one-transaction and a two-transaction IRS correction — you do not model that yourself.

Follow the chain:
- the new document carries `correctedFromId` → the original `documentId`
- the original now carries `correctedById` → the new `documentId`
- the new document's `formFields.isCorrected` is `true`

## Voiding

`form1099NecVoid` — `POST /v4/documents/1099-nec/{documentId}/void`, with an `Idempotency-Key`.
No body required.

Same pattern: a new document is filed with `voidedFromId` pointing back, the original gets
`voidedById`, and `formFields.isVoid` is `true` so the void checkbox is marked on the PDF.

## Use correction vs. void

- Wrong **amount, address or name** on an otherwise legitimate return → **correct**.
- The return **should never have been filed** at all (wrong payee entirely, duplicate filing) →
  **void**.

## After the amendment

Both operations return the new document with fresh `payerUrl` / `payeeUrl` PDFs. Mail the payee
their corrected copy with `form1099NecMail` against the **new** `documentId`. Listen for
`TEN99_NEC_CORRECTED` and `TEN99_NEC_VOIDED`, then `TEN99_NEC_ACCEPTED` / `TEN99_NEC_REJECTED` for
the authorities' response to the amendment.

## Finding what needs amending

`form1099NecList` (`GET /v4/documents/1099-nec`) filters on `status`, `filingYear`, `userId`,
`payeeTinFingerprint` and `payerTinFingerprint`, paged by `page` at up to 100 records. Listing
`status=REJECTED` for a filing year is the practical way to build a remediation queue.
