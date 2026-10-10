# Examples

All examples assume:

```bash
export HFAI_API_KEY="hfai_live_…"
export HFAI_BASE="https://api.heartflowai.net"
```

## curl — non-streaming

```bash
curl "$HFAI_BASE/v1/chat/completions" \
  -H "Authorization: Bearer $HFAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "deepseek-v4.1-flash",
    "messages": [
      { "role": "system", "content": "You are concise." },
      { "role": "user", "content": "Explain HFAI Cloud in one sentence." }
    ],
    "max_tokens": 200
  }'
```

## curl — streaming

```bash
curl -N "$HFAI_BASE/v1/chat/completions" \
  -H "Authorization: Bearer $HFAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-oss:120b",
    "messages": [{ "role": "user", "content": "Write a haiku about Solana." }],
    "stream": true
  }'
```

## Python — OpenAI SDK

```python
from openai import OpenAI

client = OpenAI(
    base_url="https://api.heartflowai.net/v1",
    api_key="hfai_live_…",
)

# Non-streaming
resp = client.chat.completions.create(
    model="deepseek-v4.1-flash",
    messages=[{"role": "user", "content": "hello"}],
)
print(resp.choices[0].message.content)

# Streaming
stream = client.chat.completions.create(
    model="deepseek-v4.1-flash",
    messages=[{"role": "user", "content": "hello"}],
    stream=True,
)
for chunk in stream:
    delta = chunk.choices[0].delta.content
    if delta:
        print(delta, end="", flush=True)
```

## Node — OpenAI SDK

```js
import OpenAI from "openai";

const client = new OpenAI({
  baseURL: "https://api.heartflowai.net/v1",
  apiKey: process.env.HFAI_API_KEY,
});

const stream = await client.chat.completions.create({
  model: "deepseek-v4.1-flash",
  messages: [{ role: "user", content: "hello" }],
  stream: true,
});

for await (const chunk of stream) {
  process.stdout.write(chunk.choices[0]?.delta?.content ?? "");
}
```

## Raw fetch (browser / edge)

```js
const res = await fetch("https://api.heartflowai.net/v1/chat/completions", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    Authorization: `Bearer ${process.env.HFAI_API_KEY}`,
  },
  body: JSON.stringify({
    model: "deepseek-v4.1-flash",
    messages: [{ role: "user", content: "hello" }],
  }),
});

const data = await res.json();
console.log(data.choices[0].message.content);
console.log("cost (micro-USD):", data.usage_ext.cost_micro);
```

## List models

```bash
curl "$HFAI_BASE/v1/models" -H "Authorization: Bearer $HFAI_API_KEY"
```

## Check balance and usage

```bash
curl "$HFAI_BASE/me"    -H "Authorization: Bearer $HFAI_API_KEY"
curl "$HFAI_BASE/usage" -H "Authorization: Bearer $HFAI_API_KEY"
```

## Create a spend-capped key

```bash
curl -X POST "$HFAI_BASE/keys" \
  -H "Authorization: Bearer $HFAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{ "name": "ci", "spendCapUsd": 25 }'
```

## Tips

- Set `max_tokens` to bound cost per request (clamped to 4096).
- Use `stream: true` for chat UIs; the final chunk carries `cost_micro`.
- Prefer a **spend-capped key** for anything that could leak.
- On `402`, top up; on `429`, back off; on `502`/`503`, retry or switch model.
