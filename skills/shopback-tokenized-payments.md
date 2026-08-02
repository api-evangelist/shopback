---
name: Link a ShopBack account and charge a returning user
description: Run the ShopBack tokenized-payments flow end to end — link session, consent, auth-code exchange, cashback balance, pre-authorization hold, capture or void, and immediate charge.
api: openapi/shopback-online-payments-openapi.yml
operations:
  - login
  - initiate-link-session
  - get-link-session
  - swap-payment-token
  - get-cashback-balance
  - create-pre-auth
  - get-pre-auth
  - capture-pre-auth
  - void-pre-auth
  - immediate-charge
  - unlink-payment-token
generated: '2026-08-02'
method: generated
source: >-
  openapi/shopback-online-payments-openapi.yml +
  https://docs.shopback.com/reference/account-linking-1 +
  https://docs.shopback.com/reference/pre-auth-requests +
  https://docs.shopback.com/reference/key-concepts
---

# Link a ShopBack account and charge a returning user

Use this when an app wants to charge a returning ShopBack user without sending
them through a full checkout redirect each time.

## Part 1 — Link the account (once per user)

1. **Authenticate** — `login` (`POST /auth/login`); use the returned merchant JWT
   as `Authorization: Bearer <merchant_jwt>` on every call below.

2. **Start a link session** — `initiate-link-session`
   (`POST /tokenized-payment/v1/link-sessions/link`) with `callbackUrl` (must be
   on your channel allowlist), `state` (a CSRF nonce you generate),
   `merchantUserId` (your stable opaque user id) and a `userHint` carrying either
   `phone` or `email`. The response returns `linkToken`, `redirectUrl`,
   `appToken` and `expiresAt` — the session TTL is 20 minutes.

3. **Open the consent page** — open `redirectUrl` in an in-app browser
   (`SFSafariViewController` on iOS, Chrome Custom Tabs on Android) passing
   `X-ShopBack-App-Token: <appToken>`. A plain WebView will not work.

4. **Handle the return** — ShopBack redirects to
   `callbackUrl?state=<state>&code=<authCode>`. Verify `state` matches what you
   sent, then forward `code` to your backend immediately — it is single-use and
   expires in 60 seconds. Use `get-link-session`
   (`GET /tokenized-payment/v1/link-sessions/link/{linkToken}`) to poll the
   session state if you need to.

5. **Exchange the code** — `swap-payment-token`
   (`POST /tokenized-payment/v1/link-sessions/token`) with `{ "code": ... }`.
   Store the returned `paymentToken` server-side against your `merchantUserId`.
   It is long-lived, reusable and revocable. Re-linking the same user returns the
   same token.

## Part 2 — Charge the user

6. **Optional: check cashback** — `get-cashback-balance`
   (`POST /tokenized-payment/v1/tokens/cashback-balance`) with `paymentToken`
   returns `cashbackBalance` and `currency`. By default (`useCashback: true`)
   ShopBack applies available cashback to reduce the net charge.

7. **Hold funds** — `create-pre-auth` (`POST /tokenized-payment/v1/pre-auths`)
   with `paymentToken`, `merchantUserId`, `amount`, `currency`, `merchantRef` and
   optional `merchantMetadata`. **Requires** a fresh UUID v4 in
   `X-ShopBack-Idempotent-Id`. The pre-auth returns `id` and `status`
   (`AUTHORIZED` on success).

8. **Settle or release**:
   - `capture-pre-auth` (`POST /tokenized-payment/v1/pre-auths/{id}/capture`)
     settles the hold and creates an order — the response carries `orderUuid`.
     Optionally pass `useCashback` and `callbackUrl`.
   - `void-pre-auth` (`POST /tokenized-payment/v1/pre-auths/{id}/void`) releases
     the hold. If the provider rejects the void the pre-auth stays `AUTHORIZED`
     and you may retry.
   - `get-pre-auth` (`GET /tokenized-payment/v1/pre-auths/{id}`) polls state
     transitions after create, capture or void.

9. **One-shot alternative** — `immediate-charge`
   (`POST /tokenized-payment/v1/charge`) authorizes and captures in one step.
   **Requires** `X-ShopBack-Idempotent-Id`. Use it only when you do not need to
   adjust the amount before capture.

10. **Disconnect** — `unlink-payment-token`
    (`POST /tokenized-payment/v1/tokens/unlink`) revokes the token. It returns
    `400` if an `AUTHORIZED` pre-auth is still open — capture or void first.

## Rules

- **Idempotency is mandatory** on `create-pre-auth` and `immediate-charge`. Reuse
  the key to retry after a timeout or 5xx; generate a new one for a deliberate
  retry after a decline.
- **Token lifecycle.** A `paymentToken` is `LINKED` until revoked. Treat it as
  valid until you receive `payment-token.invalid` (401), then prompt the user to
  re-link.
- **Do not retry 409s** — `pre-auth.already-captured`, `pre-auth.already-voided`,
  `pre-auth.declined` and `pre-auth.expired` are terminal state transitions.
- **Webhook.** After capture or immediate charge, ShopBack POSTs the order
  outcome to your `callbackUrl` with `order_uuid`, `order_status`
  (`SUCCESS`/`FAILED`) and `order_context_token`. Respond 200 fast, dedupe, and
  treat the synchronous API response as the source of truth.

## Related artifacts

- `errors/shopback-problem-types.yml` — the full tokenized-payments error slugs
- `conventions/shopback-conventions.yml`
- `data-model/shopback-data-model.yml` — LinkSession → PaymentToken → PreAuthorization
