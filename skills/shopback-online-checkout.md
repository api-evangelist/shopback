---
name: Take a ShopBack Pay online payment
description: Authenticate as a merchant, initiate a ShopBack Pay online order, redirect the shopper into ShopBack checkout, and confirm the outcome from both the order status API and the payment-notification webhook.
api: openapi/shopback-online-payments-openapi.yml
operations:
  - login
  - initiateOrder
  - getOrderInfo
generated: '2026-08-02'
method: generated
source: >-
  openapi/shopback-online-payments-openapi.yml +
  https://docs.shopback.com/docs/bespoke-implementation +
  https://docs.shopback.com/docs/checkout-redirect-flow +
  https://docs.shopback.com/docs/server-to-server-payment-notification-payment-notification-webhook
---

# Take a ShopBack Pay online payment

Use this when a merchant wants to accept ShopBack Pay or ShopBack PayLater in a
web or app checkout via the direct (bespoke) API rather than a storefront plugin.

## Before you start

- You need a Merchant ID and a Merchant Secret generated in the ShopBack for
  Business merchant portal (`https://business.shopback.sg/signin` or
  `https://business.shopback.my/signin`). Sandbox and production secrets differ.
- Base URLs: sandbox `https://integrations-sandbox.shopback.com/demo/merchant`,
  production `https://prod-merchant-service.hoolah.co/merchant`.
- All calls are HTTPS with TLS 1.2 or greater and `Content-Type: application/json`.

## Steps

1. **Authenticate** — call `login` (`POST /auth/login`) with `username`
   (Merchant ID) and `password` (Merchant Secret). The response carries `token`
   and `expiresAt`. The token is valid for 8 hours; ShopBack recommends
   generating a fresh one per transaction. Send it as
   `Authorization: Bearer <token>` on every subsequent call.

2. **Create the order** — call `initiateOrder` (`POST /order/initiate`).
   Required fields are `totalAmount`, `currency`, `items`, `billingAddress` and
   `shippingAddress`. Supply `callbackUrl` (your webhook), `returnToShopUrl`
   (success return) and `closeUrl` (failure return). Send a fresh UUID v4 in the
   `X-ShopBack-Idempotent-Id` header for this logical attempt.
   The response returns `orderContextToken`, `orderId` and `orderUuid`.

3. **Store the identifiers before redirecting.** `orderUuid` is private to you
   and is the value you will use to verify the webhook. `orderContextToken` is
   public and is what travels in the redirect.

4. **Redirect the shopper** to the ShopBack checkout with the
   `orderContextToken` — sandbox `https://demo-checkout.shopback.com/paylater`,
   production `https://checkout.shopback.sg/paylater` or
   `https://checkout.shopback.my/paylater`. ShopBack returns the shopper to your
   `returnToShopUrl` or `closeUrl`.

5. **Confirm the outcome.** Call `getOrderInfo` (`GET /order/{uuid}`) with the
   `orderUuid` — this synchronous response is the source of truth. In parallel,
   accept the payment-notification webhook POST to your `callbackUrl`
   (`order_status`, `order_uuid`, `order_context_token`, `cart_id`, and
   `failure_code` when `order_status` is `ERROR`).

## Rules

- **Idempotency.** Reuse the same `X-ShopBack-Idempotent-Id` when re-sending an
  unchanged request after a timeout, a 5xx or no response. Generate a new UUID v4
  for any new or changed order. Never construct keys by hand — a collision
  silently returns the first result.
- **Webhook verification.** There is no signature header. Compare the incoming
  `order_uuid` against the one you stored in step 3 before acting; do not trust
  `order_context_token`, which is public.
- **Webhook response.** Return HTTP 200 immediately and process asynchronously.
  A non-200 is retried for up to 30 minutes. Handle duplicate deliveries
  idempotently, and never block payment confirmation on webhook delivery.
- **Errors.** `404` means merchant, currency or order not found; `412` means an
  invalid or expired token or an invalid order status; `409` means a duplicate
  request — check your idempotency key; `500`/`502`/`503`/`504` should be retried
  with exponential backoff. Full catalog: `errors/shopback-problem-types.yml`.
- **Limits at go-live.** 1–1000 SGD/MYR unless ShopBack agrees otherwise.

## Related artifacts

- `conventions/shopback-conventions.yml` — auth, idempotency, retries
- `errors/shopback-problem-types.yml` — error catalog
- `asyncapi/shopback-payment-notification-webhooks.yml` — webhook payload
- `sandbox/shopback-sandbox.yml` — environments and certification
