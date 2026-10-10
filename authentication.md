# Authentication

The API accepts two kinds of credentials:

| Credential | Header | Who uses it |
| --- | --- | --- |
| **API key** | `Authorization: Bearer hfai_live_…` | External clients, SDKs, servers |
| **Session cookie** | `Cookie: hfai_session=…` (set automatically) | The browser app at `cloud.heartflowai.net` |

Both resolve to the same account. Use an **API key** for anything outside the browser.

## API keys

Create a key in the app (**API keys → + Create key**) or via
[`POST /keys`](api-keys.md). Send it on every request:

```
Authorization: Bearer hfai_live_…
```

Keys are:

- **Shown once** at creation, then stored only as a SHA-256 hash — we cannot show it again.
- **Revocable** at any time ([`DELETE /keys/:id`](api-keys.md)).
- Optionally **spend-capped** (`spendCapUsd` / `spendCapMicro`) so a leaked key can only
  spend up to a limit.

## Wallet session (Sign-In With Solana)

The browser app authenticates with **SIWS**:

1. `POST /auth/nonce` with your wallet address → returns a `nonce` and a `message`.
2. Sign the `message` with your wallet.
3. `POST /auth/verify` with `{ address, signature, nonce }` → the API sets an
   `HttpOnly; Secure; SameSite=None` session cookie.

The session is **never** stored in `localStorage` or any JS-readable storage. The cookie
is sent automatically on browser requests (`credentials: 'include'`). `POST /auth/logout`
clears it.

### `POST /auth/nonce`

```json
{ "address": "<base58 Solana address>" }
```

Response:

```json
{ "nonce": "a1b2c3d4e5f6g7h8", "message": "cloud.heartflowai.net wants you to sign in with your Solana account:\n…" }
```

### `POST /auth/verify`

```json
{ "address": "<base58>", "signature": "<base58 ed25519 signature>", "nonce": "<nonce>" }
```

Response (and sets the session cookie):

```json
{ "user": { "id": "…", "wallet": "…", "tier": "standard", "tierLabel": "Standard", "hfaiBalance": 0, "creditsMicro": 0, "creditsUsd": 0 } }
```

### `POST /auth/logout`

Clears the session cookie.

## CORS

Browser requests must come from an allow-listed origin. The deployed API allows
`https://cloud.heartflowai.net` (and the marketing/wallet origins). If you self-host,
add your origin to `ALLOWED_ORIGINS` in `wrangler.toml`.

State-changing requests from a browser must carry an allow-listed `Origin` header (CSRF
protection), which browsers send automatically.

## Which should I use?

- **Servers, scripts, SDKs, third-party apps → API key.**
- **The bundled web app → session cookie** (handled for you).

Never embed an API key in client-side code that ships to users.
