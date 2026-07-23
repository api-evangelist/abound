---
name: Consume Abound webhooks
description: Receive and verify Abound's 44 HMAC-signed webhook events instead of polling for TIN verification, filing acknowledgement and mail delivery.
api: openapi/abound-v4-openapi.yml
operations: []
events:
  - TIN_VERIFICATION_MATCH
  - TIN_VERIFICATION_MISMATCH
  - TEN99_NEC_ACCEPTED
  - TEN99_NEC_REJECTED
  - MAILING_DELIVERED
  - MAILING_RETURNED_TO_SENDER
status: historical
---

# Consume Abound webhooks

> **This API is retired** and no longer delivers events. The full event catalogue is preserved in
> `asyncapi/abound-webhooks-asyncapi.yml`.

Almost everything interesting in Abound is asynchronous: the IRS TIN match, the tax authorities'
acceptance of a filing, and physical mail delivery. Webhooks — not polling — are the correct way
to observe all three.

## The envelope

Every event POSTs the same base payload to your registered HTTPS endpoint:

| field | meaning |
|---|---|
| `id` | id of this delivery attempt |
| `webhookId` | id of the webhook event |
| `event` | the event name, e.g. `TEN99_NEC_ACCEPTED` |
| `timestamp` | ISO 8601 time the attempt was triggered |
| `resourceId` | id of the resource that triggered it |
| `resourceUrl` | URL of that resource |

Note the payload carries **references, not the resource itself** — fetch `resourceUrl` (or call
the matching retrieve operation) to get current state.

## Verify before you trust

Each request carries an **`Abound-Signature`** header holding an HMAC of the payload. Compute the
HMAC over the raw body with your signing secret and compare in constant time. Reject anything that
fails. Never act on an unverified event.

## Handling rules

- **Acknowledge fast.** Return `2xx` immediately and do the real work on a queue.
- **Deduplicate on `id`.** Delivery is at-least-once; the same event can arrive twice.
- **Do not assume ordering.** A `TEN99_NEC_ACCEPTED` can land before you finished processing
  `TEN99_NEC_FILED`. Re-read the resource rather than inferring state from event sequence.

## The events that matter

- **TIN verification** — `TIN_VERIFICATION_CREATED`, `_PENDING`, `_MATCH`, `_MISMATCH`.
  Gate filing on `MATCH`; queue payee remediation on `MISMATCH`.
- **1099 lifecycle** — for each of `TEN99_NEC_`, `TEN99_MISC_`, `TEN99_K_`, `TEN99_INT_`:
  `CREATED`, `FILED`, `ACCEPTED`, `REJECTED`, `CORRECTED`, `VOIDED`, `DELETED`.
  `REJECTED` is the one that needs a human — see the correct-or-void skill.
- **Mailings** — `MAILING_CREATED`, `_PROCESSING_FOR_DELIVERY`, `_IN_TRANSIT`, `_DELIVERED`,
  `_RETURNED_TO_SENDER`, `_UPDATED`, `_DELETED`. `RETURNED_TO_SENDER` means a bad payee address.
- **Forms** — `W_9_CREATED`, `W_8BEN_CREATED`, `W_8BEN_E_CREATED`.
- **Users** — `USER_CREATED`, `USER_UPDATED`.
