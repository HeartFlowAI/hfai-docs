# HFAI Cloud API

HFAI Cloud is an **OpenAI-compatible** inference gateway. Point any OpenAI SDK at it,
authenticate with an `hfai_live_…` API key (or your wallet session), and it proxies a
curated catalogue of frontier models through one endpoint.

- **Base URL:** `https://api.heartflowai.net`
- **Auth:** `Authorization: Bearer hfai_live_…`
- **Billing:** prepaid USD credits, bought with **$HFAI** at a USD peg. Pass-through
  pricing — you pay the model's listed price, no markup.

```bash
curl https://api.heartflowai.net/v1/chat/completions \
  -H "Authorization: Bearer $HFAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "deepseek-v4.1-flash",
    "messages": [{ "role": "user", "content": "hello" }]
  }'
```

## How it works

1. **Sign in** with a Solana wallet (Sign-In With Solana) at
   [cloud.heartflowai.net](https://cloud.heartflowai.net/).
2. **Top up** prepaid credits by paying **$HFAI** (Solana Pay). $10 of $HFAI = $10 of credits.
3. **Create an API key** (`hfai_live_…`) and use it from any OpenAI-compatible harness.
4. **Use it.** Every request is metered per token and deducted from your balance.

Inference is served by the operator's upstream provider using the operator's own key, which
is never exposed to clients.

## In this book

- [Getting started](getting-started.md) — key, top-up, first request.
- [Authentication](authentication.md) — API keys, wallet sessions, CORS.
- [Credits & billing](credits.md) — prepaid credits, top-up, metering.
- [Models](models.md) — the catalogue and prices.
- [Chat completions](chat-completions.md) — the main endpoint.
- [API keys](api-keys.md) — create, list, revoke.
- [Usage](usage.md) — read your metered spend.
- [Errors & rate limits](errors-and-limits.md).
- [Examples](examples.md) — curl, Python, Node.

> **Endpoint host.** All examples use `https://api.heartflowai.net`. The API is a
> Cloudflare Worker; you can also target a `*.workers.dev` host if you self-host.
