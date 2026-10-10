# Models

The catalogue is a curated set of frontier models. Every model is metered — there are no free models.

## `GET /v1/models`

Returns the model list with prices. Auth is optional: with a valid key/session you also
get your wallet price (which equals the list price — pricing is pass-through).

```bash
curl https://api.heartflowai.net/v1/models -H "Authorization: Bearer $HFAI_API_KEY"
```

```json
{
  "object": "list",
  "data": [
    {
      "id": "deepseek-v4.1-flash",
      "label": "DeepSeek V4.1 Flash",
      "context": 128000,
      "list":   { "inMicro": 300000, "outMicro": 1200000 },
      "wallet": { "inMicro": 300000, "outMicro": 1200000 },
      "saveBps": 0
    }
  ]
}
```

Prices are in **micro-USD per 1,000,000 tokens**: `300000` = `$0.30` per 1M input tokens.

## Catalogue

| Model id | Label | Context | Input / 1M | Output / 1M |
| --- | --- | --- | --- | --- |
| `deepseek-v4.1-flash` | DeepSeek V4.1 Flash | 128k | $0.30 | $1.20 |
| `gpt-oss:120b` | GPT-OSS 120B | 128k | $0.15 | $0.60 |
| `gpt-oss:20b` | GPT-OSS 20B | 128k | $0.07 | $0.30 |
| `gemma4` | Gemma 4 | 128k | $0.14 | $0.40 |
| `glm-5.3-flash` | GLM 5.3 Flash | 128k | $0.15 | $0.50 |
| `minimax-m3` | MiniMax M3 | 128k | $0.60 | $2.40 |

Prices are the model's listed price (pass-through). The list contains only models
configured on the deployment.

## Choosing a model

- **Cheapest / fastest:** `gpt-oss:20b`.
- **Balanced general use:** `deepseek-v4.1-flash`, `gpt-oss:120b`.
- **Long context / reasoning:** `minimax-m3`.

Use the model `id` in the `model` field of a chat request.

> **Note:** model ids are the gateway's ids, not the upstream provider's. Send the `id`
> exactly as returned by `/v1/models`.
