---
name: Accept a ShopBack Pay in-store QR payment
description: Sign an SB1-HMAC-SHA256 request, create a merchant-presented or customer-presented QR order at the point of sale, confirm the outcome, and refund or cancel it.
api: openapi/shopback-in-store-payments-openapi.yml
operations:
  - Create dynamic QR order
  - Scan consumer QR
  - Get order status
  - Order refund
  - Cancel order
  - Notification
generated: '2026-08-02'
method: generated
source: >-
  openapi/shopback-in-store-payments-openapi.yml +
  https://docs.shopback.com/reference/in-store-getting-started +
  https://docs.shopback.com/reference/generating-hmac-signature +
  https://docs.shopback.com/docs/pos-api-troubleshooting-list
---

# Accept a ShopBack Pay in-store QR payment

Use this for point-of-sale checkout — counter, terminal, kiosk — and for
customer-facing apps and websites that pay in an in-store context.

## Before you start

- ShopBack issues an `accessKeyId`, an `accessKeySecret` and one `posId` per
  payment surface (terminal, register, kiosk, app or website instance). Sandbox
  and production credentials differ.
- Base URLs: sandbox `https://integrations-sandbox.shopback.com/posi-sandbox`,
  production `https://integrations.shopback.sg/posi` (Singapore) or
  `https://integrations.shopback.com.hk/posi` (Hong Kong).
- `amount` is expressed in the smallest denomination of the currency, and the
  country/currency pair must be one ShopBack supports (`country` is ISO-3166-1
  alpha).

## Step 0 — Sign every request

Build the string to sign in this exact order, newline-separated:

```
POST
application/json
2026-08-02T02:29:33.123Z
https://integrations.shopback.sg/posi/v1/instore/order/create
<sha256 hex digest of the alphabetically key-sorted, stringified JSON body>
```

HMAC-SHA256 it with your `accessKeySecret`, hex-encode it, and send:

```
Authorization: SB1-HMAC-SHA256 <accessKeyId>:<hmacSignature>
Date: <the same ISO-8601 UTC timestamp used in the signature>
Content-Type: application/json
```

## Steps

1. **Merchant-presented QR** — `Create dynamic QR order`
   (`POST /v1/instore/order/create`) with `posId`, `country`, `amount`,
   `currency`, a unique `referenceId`, `qrType` (e.g. `base64`), optional
   `partner` and `orderMetadata`, and your `webhookUrl`. Send a UUID v4 in
   `X-ShopBack-Idempotent-Id`. Render the returned `qrCode` on your display, or
   send the customer to the returned URL for the redirect flow.

2. **Customer-presented QR** — `Scan consumer QR`
   (`POST /v1/instore/order/scan`) with the same fields plus
   `consumerQrPayload` read from the customer's ShopBack app.

3. **Confirm the outcome** — poll `Get order status`
   (`GET /v1/instore/order/{referenceId}`) and/or accept the
   `Notification` webhook ShopBack POSTs to your `webhookUrl`
   (`traceId`, `shopbackOrderId`, `status`, `orderAmount`, `refundAmount`,
   `failureReason`, `referenceId`, `currency`, `posId`, `paymentType`). Return
   HTTP 200 with the `referenceId`.

4. **Refund** — `Order refund`
   (`POST /v1/instore/order/{referenceId}/refund`) with `amount`, `reason`,
   `referenceId`, `posId` and optional `refundMetadata`. The order must be in the
   captured state.

5. **Cancel** — `Cancel order`
   (`POST /v1/instore/order/{referenceId}/cancel`) with a `reason`, before the
   payment is processed. Cancellation is not allowed once the customer has
   swiped to pay in the ShopBack app.

## Rules

- **`referenceId` must be unique.** Reusing one returns `409 Invalid
  ReferenceId. ReferenceId provided is already assigned to an existing resource.`
- **HMAC failures are almost always payload handling.** `401 Invalid signature`
  means one of: wrong environment credentials, body keys not alphabetically
  sorted, body stringified incorrectly or twice, wrong `Content-Type`, or a
  base64 instead of hex digest.
- **Timestamps must be UTC** — `2026-08-02T08:40:47.000Z`, never `+08:00`. Clock
  skew of about a minute expires the signature.
- **Refund and cancel require the captured state**; against a pending or
  abandoned order they return `409 Order is not in captured state.`
- Every response and error carries a `traceId` — log it, and quote it to ShopBack
  integration support.

## Related artifacts

- `errors/shopback-error-codes.yml` — the full in-store troubleshooting catalog
- `conventions/shopback-conventions.yml` — HMAC signing and idempotency
- `asyncapi/shopback-payment-notification-webhooks.yml`
- `sandbox/shopback-sandbox.yml` — environments, credentials and the Postman collection
