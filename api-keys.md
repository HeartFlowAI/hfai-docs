# API keys

Manage gateway tokens. A key proxies the operator's providers — the caller never sees an
upstream key. Any signed-in user may create one; usage is still metered against credits,
so a key without credits cannot spend.

## `GET /keys`

List your keys (masked) and whether you may create one.

```bash
curl https://api.heartflowai.net/keys -H "Authorization: Bearer $HFAI_API_KEY"
```

```json
{
  "canCreate": true,
  "data": [
    { "id": "…", "name": "prod", "prefix": "hfai_live", "last4": "a1b2", "createdAt": 1791645398000, "lastUsedAt": 1791645400000 }
  ]
}
```

## `POST /keys`

Create a key. **The full key is returned once** — store it immediately.

```json
{ "name": "prod", "spendCapUsd": 25 }
```

| Field | Type | Notes |
| --- | --- | --- |
| `name` | string | Optional label. Default `default`. |
| `spendCapUsd` | number | Optional USD spend cap for this key. |
| `spendCapMicro` | number | Optional cap in micro-USD (alternative to `spendCapUsd`). |

Response:

```json
{
  "key": "hfai_live_…",
  "id": "…",
  "name": "prod",
  "prefix": "hfai_live",
  "last4": "a1b2",
  "createdAt": 1791645398000,
  "notice": "store this key now; it will not be shown again"
}
```

## `DELETE /keys/:id`

Revoke a key.

```bash
curl -X DELETE https://api.heartflowai.net/keys/<id> \
  -H "Authorization: Bearer $HFAI_API_KEY"
```

```json
{ "ok": true }
```

## Notes

- Keys are stored **hashed** (SHA-256); we cannot recover a lost key — create a new one.
- Key creation is rate-limited to **10 per hour** per user.
- A per-key **spend cap** is enforced before each request; a request whose worst-case cost
  exceeds the remaining cap is rejected (`402`/`403`).
