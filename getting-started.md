# Getting started

Five minutes from zero to your first streamed reply.

## 1. Sign in

Go to **[cloud.heartflowai.net](https://cloud.heartflowai.net/)** and click **Connect
Phantom** (or **Use HeartFlow Wallet**). You sign a message with your Solana wallet — no
email, no password. This creates a session so you can manage keys and credits.

## 2. Add credits

Open **Credits** and pick an amount ($10 / $25 / $50 / $100, or a custom $1–$1000). Your
wallet pops up to approve a **$HFAI** transfer; credits are applied automatically once
the payment confirms. See [Credits & billing](credits.md).

## 3. Create an API key

Open **API keys** → **+ Create key**. Copy the key now — it is shown **once** and stored
only as a hash. It looks like `hfai_live_…`.

## 4. Make a request

The API is OpenAI-compatible, so any OpenAI client works. Replace the base URL and key:

```bash
export HFAI_API_KEY="hfai_live_…"

curl https://api.heartflowai.net/v1/chat/completions \
  -H "Authorization: Bearer $HFAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "deepseek-v4.1-flash",
    "messages": [{ "role": "user", "content": "hello" }],
    "stream": true
  }'
```

Or with the official Python SDK:

```python
from openai import OpenAI

client = OpenAI(
    base_url="https://api.heartflowai.net/v1",
    api_key="hfai_live_…",
)

resp = client.chat.completions.create(
    model="deepseek-v4.1-flash",
    messages=[{"role": "user", "content": "hello"}],
)
print(resp.choices[0].message.content)
```

## 5. Check your balance

Every request is metered. See your remaining credits and daily spend:

```bash
curl https://api.heartflowai.net/me -H "Authorization: Bearer $HFAI_API_KEY"
curl https://api.heartflowai.net/usage -H "Authorization: Bearer $HFAI_API_KEY"
```

## Next

- [Models](models.md) — pick a model and see prices.
- [Chat completions](chat-completions.md) — full request/response reference.
- [Errors & rate limits](errors-and-limits.md) — what the codes mean.
