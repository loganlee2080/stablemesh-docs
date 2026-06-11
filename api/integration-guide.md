# Integration Guide

This is a start-to-finish walkthrough for integrating the Stable Mesh **Open API** — from your first authenticated call to issuing a live card and receiving signed webhooks. Every example is copy-paste `curl`; swap in your own key and base URL.

If you want per-endpoint detail, see the reference pages: [Authentication](authentication.md), [Wallet & Funding](wallet.md), [Cards](cards.md), [Transactions](transactions.md), [Idempotency](idempotency.md), [Webhooks](webhooks.md), [Errors](errors.md).

---

## 1. Before you start

You need an **API key**, issued by the Stable Mesh team during onboarding. A key looks like `smk_…`; it is shown **once** at creation and stored only as a hash on our side — save it in your secret manager. If you lose it, ask for a rotation.

| Environment | Base URL | Use for |
|---|---|---|
| Sandbox / Dev | `https://api-server-dev-1440.up.railway.app` | build & test |
| Production | `https://api.stablemesh.io` | Live cards and real funds. |

All endpoints are `POST`, live under `/v1`, take a JSON body, and require two headers:

```
Content-Type: application/json
X-API-KEY: smk_your_key_here
```

> **Tip — set up your shell once:**
> ```bash
> export SM_BASE="https://api-server-dev-1440.up.railway.app"
> export SM_KEY="smk_your_key_here"
> sm() { curl -s -X POST "$SM_BASE$1" -H "Content-Type: application/json" -H "X-API-KEY: $SM_KEY" -d "${2:-{} }"; }
> ```
> Then every step below is just `sm /v1/...`.

---

## 2. The mental model

- **One account, one balance.** Your account holds a single **USDT** balance. You top it up by depositing USDT on-chain.
- **Cards draw from that balance.** Creating or funding a card **deducts USDT** from your balance and credits the card (denominated in **USD**). A card can never spend more than you funded it.
- **Fees are taken at the card step, not on deposit.** When you create or top up a card, a **deposit fee** (a percentage, set per-account) and an optional **card-open fee** (set per-BIN) are deducted. The formula is:
  ```
  card_credit = amount − (amount × deposit_fee_rate) − open_fee
  ```
  Example at the default 2% deposit fee, 0 open fee: create with `amount=50` → card is credited **49**. At a negotiated **0%** rate: card is credited the full **50**.

---

## 3. Authenticate — verify your key

List your balances. This is the simplest authenticated call and confirms your key works.

```bash
sm /v1/wallet/assetList
```

```json
{
  "code": 0,
  "message": "ok",
  "assetList": [
    { "currency": "USDT", "availableAmount": "1000", "frozenAmount": "0", "assetType": "CRYPTO", "status": "ACTIVE" },
    { "currency": "USDC", "availableAmount": "0",    "frozenAmount": null, "assetType": "CRYPTO", "status": null }
  ]
}
```

**Auth failure modes** (handle both):

| Situation | HTTP | Body |
|---|---|---|
| No `X-API-KEY` header | `403` | _(empty)_ |
| Invalid / revoked key | `401` | `{ "code": 11002, "message": "Invalid or revoked API key" }` |

---

## 4. Fund your account

Get your deposit address, then send USDT to it on-chain. Funds are credited automatically once the transfer confirms.

```bash
sm /v1/wallet/getAddress '{"currency":"USDT","network":"TRX"}'
```

```json
{ "code": 0, "message": "ok", "address": "T….", "network": "TRX" }
```

Poll `/v1/wallet/assetList` until your balance reflects the deposit. (In **sandbox**, the Stable Mesh team can seed a test balance for you so you can skip the on-chain step.)

---

## 5. Pick a card BIN

List the BINs available to your account, with their fees and limits.

```bash
sm /v1/card/binList
```

```json
{
  "code": 0, "message": "ok",
  "availableCardBinList": [
    {
      "id": "VISA-EFI-454924", "title": "Ether Card", "bin": "454924", "cardNetwork": "Visa",
      "fee": { "deposit": "0.02", "fx": "0.006", "card": "0", "atm": "0.02" },
      "limit": { "spendDaily": "100000", "minimalDeposit": "1" },
      "enabled": true
    }
  ]
}
```

The `fee.deposit` is the deposit-fee rate and `fee.card` is the per-card open fee. Use a BIN's `id` as the `binId` when you create a card.

---

## 6. Issue a card

```bash
sm /v1/card/create '{"amount":50,"currency":"USDT","binId":"VISA-EFI-454924"}'
```

```json
{
  "cardId": "2026060117803181873402",
  "cardStatus": "ACTIVE",
  "maskedCardNo": "2222",
  "binId": "VISA-EFI-454924",
  "openFee": 0
}
```

This deducted `50` USDT (gross) from your balance and credited the card net of fees. Confirm:

```bash
sm /v1/card/asset '{"cardId":"2026060117803181873402"}'   # → availableAmount: "49" (or "50" at 0% deposit fee)
sm /v1/wallet/assetList                                     # → USDT now 950
```

`POST /v1/card/list` returns all your cards; `POST /v1/card/detail {"cardId":…}` returns full metadata including the BIN config.

---

## 7. Top up a card

```bash
sm /v1/card/deposit '{"cardId":"2026060117803181873402","amount":100}'
```

```json
{ "code": 0, "message": "ok" }
```

The `100` USDT is debited from your balance **synchronously**; the card credit settles a few seconds later via the funding pipeline. Re-read `card/asset` until it reflects the new balance (e.g. `49 → 147` at 2%, or `50 → 150` at 0%).

---

## 8. Reveal card details (PAN / CVV / expiry)

```bash
sm /v1/card/sensitiveInfo '{"cardId":"2026060117803181873402"}'
```

Returns the card number, CVV, and expiry for display to the cardholder.

{% hint style="warning" %}
**PCI:** treat this response as cardholder data. Render it client-side, never log it, and never persist the PAN/CVV. Stable Mesh stores only a masked PAN (`************<last4>`).
{% endhint %}

---

## 9. Manage card state

```bash
sm /v1/card/freeze   '{"cardId":"2026060117803181873402"}'   # → status FROZEN
sm /v1/card/unfreeze '{"cardId":"2026060117803181873402"}'   # → status ACTIVE
sm /v1/card/cancel   '{"cardId":"2026060117803181873402"}'   # → status CANCELLED (terminal)
```

---

## 10. Read transactions

```bash
sm /v1/bill/list '{"cardId":"2026060117803181873402","pageNumber":1,"pageSize":10}'
```

```json
{ "code": 0, "message": "ok", "totalElements": 0, "totalPages": 0, "last": true, "transactions": [] }
```

---

## 11. Idempotency (do this for every mutating call)

Network retries can double-charge. Send an **`Idempotency-Key`** header (any unique string, e.g. a UUID) on every `create` / `deposit` / state-change call. Reuse the **same key** when you retry the same logical operation.

```bash
curl -s -X POST "$SM_BASE/v1/card/deposit" \
  -H "Content-Type: application/json" -H "X-API-KEY: $SM_KEY" \
  -H "Idempotency-Key: 3f1c…-your-uuid" \
  -d '{"cardId":"2026060117803181873402","amount":100}'
```

| Behaviour | Result |
|---|---|
| First request with a key | Executes normally. |
| Retry, **same key + same body** | Original response replayed; **not executed twice**. Response carries `Idempotent-Replayed: true`. |
| Same key, **different body** | `409` · `{ "code": 4211, … }` — nothing executes. |
| Two requests racing on the same key | One wins; the other gets `409` · `{ "code": 4212, "message": "…already in progress" }`. |

Keys are retained 24h. See [Idempotency](idempotency.md).

---

## 12. Webhooks — receive events

Rather than polling, register an HTTPS endpoint and Stable Mesh will push signed events to it.

### 12.1 Register your endpoint

```bash
sm /v1/webhook '{"url":"https://yourapp.example.com/sm/webhook","enabled":true,"deliver3dsCode":false}'
```

```json
{ "configured": true, "url": "https://yourapp.example.com/sm/webhook",
  "enabled": true, "secretPrefix": "whsec_IuVlLxpb",
  "secret": "whsec_IuVlLxpbI-omIBRfDaLsvJSZWHBll59v" }
```

The **`secret`** (`whsec_…`) is returned **once** — store it; you verify every delivery with it. `GET /v1/webhook` shows the config (prefix only); `POST /v1/webhook/rotate-secret` rotates it; `DELETE /v1/webhook` stops deliveries.

### 12.2 Event types

`card.created` · `card.frozen` · `card.unfrozen` · `card.cancelled` · `card.3ds.requested` · `transaction.created` · `transaction.updated`

Each delivery carries headers: `StableMesh-Event-Id`, `StableMesh-Event-Type`, and `StableMesh-Signature`.

### 12.3 Verify the signature (required)

The signature header is `StableMesh-Signature: t=<unix_seconds>,v1=<hex_hmac_sha256>`. The signed payload is the string `"<t>.<raw_request_body>"`, HMAC-SHA256'd with your `whsec_` secret. Verify over the **raw bytes** of the body, before JSON parsing.

**Python (Flask):**
```python
import hmac, hashlib, time

SECRET = "whsec_IuVlLxpbI-omIBRfDaLsvJSZWHBll59v"

def verify(raw_body: bytes, header: str, tolerance=300) -> bool:
    parts = dict(p.split("=", 1) for p in header.split(",") if "=" in p)
    t, v1 = parts.get("t"), parts.get("v1")
    if not t or not v1 or abs(time.time() - int(t)) > tolerance:
        return False
    expected = hmac.new(SECRET.encode(), f"{t}.{raw_body.decode()}".encode(), hashlib.sha256).hexdigest()
    return hmac.compare_digest(expected, v1)
```

**Node (Express, `express.raw({type: '*/*'})`):**
```js
const crypto = require("crypto");
const SECRET = "whsec_IuVlLxpbI-omIBRfDaLsvJSZWHBll59v";

function verify(rawBody, header, tolerance = 300) {
  const parts = Object.fromEntries(header.split(",").map(p => p.split("=")));
  if (Math.abs(Date.now() / 1000 - Number(parts.t)) > tolerance) return false;
  const expected = crypto.createHmac("sha256", SECRET).update(`${parts.t}.${rawBody}`).digest("hex");
  return crypto.timingSafeEqual(Buffer.from(expected), Buffer.from(parts.v1));
}
```

### 12.4 Respond and rely on retries

Return **HTTP 2xx** quickly (do heavy work async). Any non-2xx (or a timeout) is **retried with backoff**, so your handler must be **idempotent** — dedupe on `StableMesh-Event-Id`.

---

## 13. Going-to-production checklist

- [ ] Key stored in a secret manager; never in source or logs.
- [ ] `Idempotency-Key` sent on every create / deposit / state-change.
- [ ] Webhook endpoint verifies the signature over the raw body and rejects bad/stale signatures.
- [ ] Webhook handler is idempotent (dedupe on event id) and returns 2xx fast.
- [ ] PAN/CVV from `sensitiveInfo` rendered client-side, never logged or stored.
- [ ] Errors handled by `code` (see below), not just HTTP status.
- [ ] Switched `SM_BASE` to `https://api.stablemesh.io` and re-tested with a production key.

---

## 14. Error reference

Errors return a non-zero `code` in the JSON body. The most common:

| `code` | Meaning |
|---|---|
| `11002` | Invalid or revoked API key (HTTP 401) |
| `400` | Illegal / missing parameter |
| `2000` | Invalid or missing address |
| `4009` | Card out of inventory |
| `4010` | Card provisioning in progress — retry shortly (no funds were moved) |
| `4210` / `4211` / `4212` | Idempotency-Key required / mismatched / in progress |

Full list: [Errors](errors.md).

---

## 15. Quick reference — all endpoints

| Endpoint | Body | Purpose |
|---|---|---|
| `POST /v1/wallet/assetList` | `{}` | List balances |
| `POST /v1/wallet/getAddress` | `{currency, network}` | Deposit address |
| `POST /v1/card/binList` | `{}` | Available BINs + fees |
| `POST /v1/card/create` | `{amount, currency, binId}` | Issue a card |
| `POST /v1/card/list` | `{}` | List cards |
| `POST /v1/card/detail` | `{cardId}` | Card metadata |
| `POST /v1/card/asset` | `{cardId}` | Card balance |
| `POST /v1/card/deposit` | `{cardId, amount}` | Top up a card |
| `POST /v1/card/sensitiveInfo` | `{cardId}` | Reveal PAN/CVV/expiry |
| `POST /v1/card/freeze` · `unfreeze` · `cancel` | `{cardId}` | Card state |
| `POST /v1/bill/list` | `{cardId, pageNumber, pageSize}` | Transactions |
| `GET·PUT·DELETE /v1/webhook`, `POST /v1/webhook/rotate-secret` | — | Webhook config |

{% hint style="info" %}
Need a sandbox account or your first key? Contact your Stable Mesh account manager.
{% endhint %}
