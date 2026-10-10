# Usage

Read your balance, metered spend, and saved conversations.

## `GET /me`

Your wallet and credits.

```bash
curl https://api.heartflowai.net/me -H "Authorization: Bearer $HFAI_API_KEY"
```

```json
{
  "user": {
    "id": "…",
    "wallet": "<base58>",
    "hfaiBalance": 762700,
    "creditsMicro": 10000000,
    "creditsUsd": 10
  }
}
```

`creditsUsd` is your spendable balance. `hfaiBalance` is your on-chain `$HFAI` holding
(informational).

## `GET /usage`

Your balance plus a daily series (last 14 days) for a chart.

```bash
curl https://api.heartflowai.net/usage -H "Authorization: Bearer $HFAI_API_KEY"
```

```json
{
  "creditsMicro": 10000000,
  "days": [
    { "day": "2026-10-09", "costMicro": 44400, "tokens": 46 },
    { "day": "2026-10-10", "costMicro": 120000, "tokens": 2100 }
  ]
}
```

| Field | Meaning |
| --- | --- |
| `costMicro` | USD micro-units spent that day. |
| `tokens` | Prompt + completion tokens that day. |

## Conversations

Save and reload chat threads (used by the bundled app; optional for API clients).

| Method | Path | Use |
| --- | --- | --- |
| `GET` | `/conversations` | List your conversations. |
| `POST` | `/conversations` | Create one (`{ title?, persona? }`). |
| `GET` | `/conversations/:id` | Load a conversation with its messages. |
| `PATCH` | `/conversations/:id` | Rename / change persona. |
| `DELETE` | `/conversations/:id` | Delete. |

Send a conversation's `id` as `conversationId` in a [chat request](chat-completions.md) to
persist turns and auto-title the thread.

## `GET /health`

Uptime probe — no auth. Returns only a liveness flag.

```json
{ "ok": true }
```
