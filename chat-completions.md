# Chat completions

`POST /v1/chat/completions` — OpenAI-compatible chat, streaming or not.

```
Authorization: Bearer hfai_live_…
Content-Type: application/json
```

## Request body

| Field | Type | Notes |
| --- | --- | --- |
| `model` | string | A model `id` from [`/v1/models`](models.md). Defaults to the first model. |
| `messages` | array | **Required.** `[{ "role": "system"｜"user"｜"assistant", "content": "…" }]`. Max 200 messages, 200k chars total. |
| `stream` | boolean | `true` → Server-Sent Events. Default `false`. |
| `temperature` | number | Passed through to the model. |
| `max_tokens` | number | Output cap, clamped to **[1, 4096]**. Also accepts `max_completion_tokens`. |
| `persona` | string | Optional. One of `aurora`, `gigi`, `nova`, `clara` — injects that sister's system prompt. |
| `conversationId` | string | Optional. Persists the turn and auto-titles the conversation (see [Usage](usage.md)). |

## Non-streaming response

```json
{
  "id": "chatcmpl-…",
  "object": "chat.completion",
  "created": 1791645398,
  "model": "deepseek-v4.1-flash",
  "choices": [
    { "index": 0, "message": { "role": "assistant", "content": "…" }, "finish_reason": "stop" }
  ],
  "usage": { "prompt_tokens": 12, "completion_tokens": 34, "total_tokens": 46 },
  "usage_ext": { "cost_micro": 44400, "conversation_id": null }
}
```

`usage_ext.cost_micro` is what was deducted from your balance (micro-USD).

## Streaming response

With `stream: true` the API returns `text/event-stream`:

```
data: {"id":"chatcmpl-…","object":"chat.completion.chunk","choices":[{"delta":{"content":"Hel"},"index":0}]}

data: {"id":"chatcmpl-…","object":"chat.completion.chunk","choices":[{"delta":{"content":"llo"},"index":0}]}

data: {"id":"chatcmpl-…","object":"chat.completion.chunk","choices":[{"delta":{},"finish_reason":"stop","index":0}],"usage":{"prompt_tokens":12,"completion_tokens":34,"cost_micro":44400,"conversation_id":null}}

data: [DONE]
```

- Each chunk is a `chat.completion.chunk`; concatenate `choices[0].delta.content`.
- The **final** chunk carries `usage` (with `cost_micro`) and `finish_reason: "stop"`.
- The stream ends with `data: [DONE]`.
- If the upstream fails mid-stream, a chunk with `{"error": {"message": "…", "code": "upstream_error"}}` is sent.

## Personas

Send `persona` to have the API prepend that sister's system prompt:

| `persona` | Character |
| --- | --- |
| `aurora` | Connection & tool use |
| `gigi` | Market intelligence (never financial advice) |
| `nova` | Execution & next actions |
| `clara` | Clarity & regulation |

This is a convenience for the bundled app — external clients can ignore it and send their
own `system` message.

## Persisting conversations

If you send `conversationId`, the API stores the user turn, sets the conversation's model,
and auto-titles it on the first message. Manage conversations with the
`/conversations` endpoints (list, create, load, rename, delete). See
[Usage](usage.md#conversations).

## Tool calling

The API supports OpenAI **function calling** end-to-end, so any OpenAI-compatible harness can
run a tool loop. Send `tools` (and optionally `tool_choice`); the model may reply with
`tool_calls`; send the results back as `role: "tool"` messages.

### 1. Send tools

```json
{
  "model": "deepseek-v4.1-flash",
  "messages": [{ "role": "user", "content": "What is the weather in Tokyo?" }],
  "tools": [
    {
      "type": "function",
      "function": {
        "name": "get_weather",
        "description": "Get the current weather for a city",
        "parameters": {
          "type": "object",
          "properties": { "city": { "type": "string" } },
          "required": ["city"]
        }
      }
    }
  ],
  "tool_choice": "auto"
}
```

### 2. The model asks for a tool

```json
{
  "choices": [{
    "index": 0,
    "message": {
      "role": "assistant",
      "content": null,
      "tool_calls": [
        { "id": "call_0_getweather", "type": "function",
          "function": { "name": "get_weather", "arguments": "{\"city\":\"Tokyo\"}" } }
      ]
    },
    "finish_reason": "tool_calls"
  }]
}
```

`arguments` is a JSON **string** (parse it). `finish_reason` is `"tool_calls"`.

### 3. Send the tool result back

```json
{
  "model": "deepseek-v4.1-flash",
  "messages": [
    { "role": "user", "content": "What is the weather in Tokyo?" },
    { "role": "assistant", "content": null, "tool_calls": [ /* the call above */ ] },
    { "role": "tool", "tool_call_id": "call_0_getweather", "content": "{\"tempC\":22,\"condition\":\"sunny\"}" }
  ],
  "tools": [ /* same tools */ ]
}
```

The model then returns a normal assistant message with `finish_reason: "stop"`.

### Streaming

With `stream: true`, tool calls arrive as `delta.tool_calls` chunks (accumulate by `index`),
and the final chunk carries `finish_reason: "tool_calls"`:

```
data: {"choices":[{"index":0,"delta":{"tool_calls":[{"index":0,"id":"call_0_getweather","type":"function","function":{"name":"get_weather","arguments":"{\"city\":\"Tokyo\"}"}}]}}]}

data: {"choices":[{"index":0,"delta":{},"finish_reason":"tool_calls"}],"usage":{…}}
```

### Notes

- `tool_choice`: `"auto"` (default), `"none"`, `"required"`, or `{ "type": "function", "function": { "name": "…" } }`.
- Tool-call turns are metered like any other turn.
- Tool support depends on the model. Verified working on `deepseek-v4.1-flash`, `gpt-oss:120b`, and `minimax-m3`.

## Billing

Every request pre-authorizes a worst-case cost, then settles to the actual usage. A
request needs a positive balance; otherwise it returns `402 insufficient_credits`. See
[Credits & billing](credits.md).
