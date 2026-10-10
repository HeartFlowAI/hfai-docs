# Credits & billing

HFAI Cloud is **prepaid**. You buy USD credits up front by paying **$HFAI**, and every
request is metered per token and deducted from your balance.

- **Peg:** $10 of $HFAI = $10 of credits (one-way; no redemption).
- **Pricing:** pass-through — you pay the model's listed price. **No markup, no tiers.**
- **Minimum top-up:** **$1**.
- **Balance unit:** USD micro-units (`creditsMicro`; `1_000_000` = $1.00).

## Topping up

In the app: **Credits → pick an amount → Top up with $HFAI**. Your wallet signs a
Token-2022 `$HFAI` transfer and credits are applied automatically once confirmed.

Under the hood this is two calls:

### `POST /billing/topup`

Creates a Solana Pay intent and quotes the USD amount in `$HFAI`.

```json
{ "amount": "10" }
```

Response:

```json
{
  "intent": {
    "reference": "<unique reference pubkey>",
    "url": "solana:<treasury>?spl-token=<mint>&amount=…&reference=…",
    "tokenAmount": "634090.13",
    "tokenAmountRaw": "634090131696",
    "decimals": 6,
    "mint": "<$HFAI mint>",
    "recipient": "<treasury>",
    "network": "mainnet-beta",
    "usdAmount": "10",
    "price": 0.00001577,
    "expiresAt": 1791645398006,
    "blockhash": "<recent blockhash>",
    "lastValidBlockHeight": 433335023
  }
}
```

| Field | Meaning |
| --- | --- |
| `tokenAmount` | Quoted amount in UI units (decimal string). |
| `tokenAmountRaw` | Quoted amount in **base units** (integer string, exact BigInt math). |
| `decimals` | Mint decimals used for `tokenAmountRaw`. |
| `usdAmount`, `price` | The USD quote and the `$HFAI`/USD price used. |
| `expiresAt` | Quote expiry (ms since epoch). |
| `blockhash` / `lastValidBlockHeight` | Recent blockhash to build the transaction against. |
| `reference` | Unique reference key that ties the on-chain payment to this intent. |

### `POST /billing/verify`

Confirms the on-chain payment and credits the USD value.

```json
{ "reference": "<reference>", "signature": "<tx signature>" }
```

`signature` is optional — if omitted, the server looks it up on-chain from the
`reference`. Response:

```json
{ "ok": true, "creditsMicro": 10000000, "user": { "…": "…" } }
```

The server re-checks the reference, the treasury delta, the quote, replay and expiry
before crediting, so a tampered client can only harm itself.

> **Building the transaction yourself?** The intent is a standard Solana Pay request for
> a **Token-2022** mint: transfer exactly `tokenAmountRaw` base units to the treasury's
> associated token account, include the `reference` as a read-only non-signer account,
> and sign with the payer. The bundled app does this in the browser.

## How metering works

Each request pre-authorizes a worst-case cost, streams, then settles to the **actual**
token usage reported by the provider. The charge is:

```
cost_micro = (prompt_tokens × inMicro + completion_tokens × outMicro) / 1_000_000
```

where `inMicro` / `outMicro` are the model's per-1M-token prices in micro-USD (see
[Models](models.md)). The cost is returned on every response (`usage_ext.cost_micro`).

- A request with **no credits** returns `402 insufficient_credits`.
- Per-key **spend caps** (if set) are enforced before the request is served.
