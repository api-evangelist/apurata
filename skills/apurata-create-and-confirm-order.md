---
name: Create and confirm an aCuotaz installment order
description: Guide an agent to create an aCuotaz BNPL order for a purchase, track it to approval and funding, then confirm it so the merchant fulfills.
api: openapi/apurata-acuotaz-pos-openapi.yml
operations: [getLandingConfig, createOrder, getOrder, confirmOrder]
---

# Create and confirm an aCuotaz installment order

Use the Apurata aCuotaz POS REST API (base URL `https://apurata.com`). Authenticate every
call with `Authorization: Bearer <secret_token>` over HTTPS.

## Steps

1. **Check funding limits** — call `getLandingConfig` (`GET /pos/client/landing_config`).
   Confirm the purchase `amount` is between `min_amount` and `max_amount` and the
   installment count fits `min_steps`..`max_steps`.

2. **Create the order** — call `createOrder` (`POST /pos/order/create`) with `amount`,
   a merchant-unique `order_id`, `pos_client_id`, and the four required redirect URLs
   (`url_redir_on_canceled`, `url_redir_on_rejected`, `url_redir_on_success`,
   `url_redir_on_downpayment`). Creation is **idempotent on `order_id`** — resubmitting
   the same id returns the existing order with `status: already_created`, so it is safe
   to retry. Redirect the buyer to the returned `redirect_to` URL.

3. **Track status** — call `getOrder` (`GET /pos/order/{order_id}`) or listen for
   webhooks. The order moves `created -> approved -> onhold -> validated -> funded`.
   Only act on a fully `funded` order.

4. **Confirm** — once `funded`, call `confirmOrder` (`POST /pos/order/{order_id}/confirm`).
   A `200` with `modified_orders: 1` means the order is confirmed and the merchant can
   fulfill the purchase.

## Rules

- HTTPS is mandatory; never send the token over plain HTTP.
- Handle `401` (bad/missing token), `400` (validation or wrong order state), and `404`
  (unknown order) per `errors/apurata-problem-types.yml`.
- Timestamps are UTC-0; Peru local time is UTC-5.
