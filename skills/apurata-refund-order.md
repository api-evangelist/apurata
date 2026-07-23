---
name: Refund an aCuotaz order
description: Guide an agent to issue a total or partial refund against a funded aCuotaz order safely and idempotently.
api: openapi/apurata-acuotaz-pos-openapi.yml
operations: [getOrder, totalRefundOrder, partialRefundOrder]
---

# Refund an aCuotaz order

Use the Apurata aCuotaz POS REST API (base URL `https://apurata.com`) with
`Authorization: Bearer <secret_token>` over HTTPS.

## Steps

1. **Inspect the order** — call `getOrder` (`GET /pos/order/{order_id}`) and confirm the
   order is in a refundable state (funded/disbursed). Note the `amount`.

2. **Generate an idempotency token** — create a unique value for the
   `X-Unique-Token` header. This guards against duplicate refunds; reuse the same
   token if you must retry the exact same refund.

3. **Issue the refund**
   - Full refund: call `totalRefundOrder`
     (`POST /pos/order/{order_id}/total-refund`) with `author` and `reason`.
     `200` returns `refund_method`, `tracking_id`, and `status` (`success` or
     `already_refunded`).
   - Partial refund: call `partialRefundOrder`
     (`POST /pos/order/{order_id}/partial-refund`) with `amount`, `author`, `reason`.
     `200` returns `status` (`success` or `decreased_debt`).

## Rules

- Always send `X-Unique-Token`; a repeated token yields `already_refunded` instead of a
  double refund.
- `400` means an amount/disbursement constraint (e.g. partial amount too large, or the
  order cannot be refunded in its current state); `404` means the order does not exist.
- See `conventions/apurata-conventions.yml` (idempotency) and
  `errors/apurata-problem-types.yml`.
