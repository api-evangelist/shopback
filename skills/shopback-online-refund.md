---
name: Refund a ShopBack Pay online order
description: Issue a full or partial refund against a completed ShopBack Pay online order and reconcile the result against order status and the merchant activity report.
api: openapi/shopback-online-payments-openapi.yml
operations:
  - login
  - getOrderInfo
  - initiateOrderRefund
generated: '2026-08-02'
method: generated
source: >-
  openapi/shopback-online-payments-openapi.yml +
  https://docs.shopback.com/reference/initiateorderrefund +
  https://docs.shopback.com/docs/online-bespoke-error-code-list +
  https://docs.shopback.com/docs/billing-and-activity-reports
---

# Refund a ShopBack Pay online order

Use this when a shopper returns some or all of an order paid with ShopBack Pay.

## Steps

1. **Authenticate** — `login` (`POST /auth/login`) with the Merchant ID and
   Merchant Secret; send the returned `token` as `Authorization: Bearer <token>`.

2. **Check the order first** — `getOrderInfo` (`GET /order/{uuid}`) with the
   `orderUuid` you stored at checkout. Confirm the order is in a refundable
   state and note the amount already refunded before you compute the new amount.

3. **Issue the refund** — `initiateOrderRefund`
   (`POST /order/{orderUuid}/refund`). For a partial refund send `amount`,
   `description`, `refundedByEmail`, an optional `webhookUrl`, and `items[]`
   carrying the `sku` of each returned line item. For a full refund send the full
   order amount. Send a fresh `X-ShopBack-Idempotent-Id` (UUID v4) for this
   refund attempt.

4. **Read the response** — the refund result carries `status`, `requestId`,
   `createdAt`, `type`, `code`, `message` and a `details` block with
   `transactionFeeAmount` and `merchantFeeAmount`. Persist `requestId` for
   reconciliation.

5. **Reconcile** — re-run `getOrderInfo` to confirm the order's refunded state,
   and cross-check against the daily merchant activity report
   (`https://docs.shopback.com/docs/activity-report-payments`).

## Rules

- **One idempotency key per logical refund.** Retrying the *same* refund after a
  timeout or 5xx must reuse the key. A genuinely new refund (a second return on
  the same order) needs a new key — reusing the old one returns the first result.
- **`409 Conflict` means duplicate refund.** Do not resend as new; look up the
  original by `requestId`.
- **`412 Precondition Failed`** on a refund usually means the refund amount is
  wrong for the order's state — re-read the order before retrying.
- Refunds are not supported against orders that were never captured; use the
  cancel path on the in-store surface for pre-capture reversal.

## Related artifacts

- `errors/shopback-problem-types.yml`
- `conventions/shopback-conventions.yml`
- `data-model/shopback-data-model.yml` — Order → Refund → PartialRefundItem
