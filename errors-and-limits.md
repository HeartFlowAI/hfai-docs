# Errors & rate limits

## Error shape

Every error is JSON:

```json
{ "error": { "message": "no credits; top up with Solana Pay", "code": "insufficient_credits" } }
```

## Status codes

| HTTP | `code` | Meaning |
| --- | --- | --- |
| `400` | `bad_request` | Malformed request (missing `messages`, bad amount, etc.). |
| `401` | `unauthorized` | Missing/invalid API key or session. |
| `402` | `insufficient_credits` | Not enough credits (or a per-key spend cap was hit). Top up. |
| `403` | `forbidden` | Not allowed (e.g. key creation blocked, CSRF origin). |
| `404` | `not_found` | Unknown model / key / conversation. |
| `429` | `rate_limited` | Too many requests — slow down. |
| `500` | `internal_error` | Unexpected server error. |
| `502` | `upstream_error` | The model provider failed. |
| `503` | `service_misconfigured` | Deployment misconfigured (e.g. treasury not set). |
| `503` | `service_unavailable` | A dependency (price oracle, RPC) is temporarily down — retry. |

## Rate limits

Per credential, sliding window:

| Scope | Limit |
| --- | --- |
| `POST /v1/chat/completions` | **60 / minute** per user |
| `POST /auth/nonce` | 30 / minute per IP |
| `POST /keys` | 10 / hour per user |
| `POST /billing/topup` | 10 / minute per user |
| `POST /billing/verify` | 20 / minute per user |

Exceeding a limit returns `429 rate_limited`. Back off and retry.

## Handling `402`

`402 insufficient_credits` means the balance (or the key's spend cap) is exhausted. Top up
in the app or via [`/billing/topup`](credits.md), then retry. A key with no credits cannot
spend — create keys freely, but fund the account to use them.

## Handling `502` / `503`

- `502 upstream_error` — the model provider failed; retry or switch model.
- `503 service_unavailable` — a transient dependency outage (price oracle or Solana RPC);
  retry shortly.

## Idempotency & safety

- A top-up `signature` can credit **at most one** intent; replaying it is rejected.
- Intents expire (~15 min); an expired intent cannot be credited.
- Verification re-checks the reference, the treasury delta and the stored quote before
  crediting, so a tampered client can only harm itself.
