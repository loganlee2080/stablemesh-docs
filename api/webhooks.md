# Webhooks

Webhooks push events to your server so you don't have to poll. Stable Mesh sends a signed `POST`
to your endpoint whenever a card transaction changes state, a 3DS/OTP challenge is raised, or a
card's lifecycle changes.

---

## Configure your endpoint

Manage your webhook with your API key under `/v1/webhook`.

| Method | Path | Purpose |
|--------|------|---------|
| `GET` | `/v1/webhook` | View current config (secret shown as prefix only) |
| `PUT` | `/v1/webhook` | Create or update the endpoint URL / options |
| `POST` | `/v1/webhook/rotate-secret` | Generate a new signing secret |
| `DELETE` | `/v1/webhook` | Remove the endpoint (stops all deliveries) |

**Create / update**

```bash
curl -X PUT https://api.stablemesh.io/v1/webhook \
  -H "X-API-KEY: $SMK_KEY" -H "Content-Type: application/json" \
  -d '{"url":"https://yourapp.example/stablemesh/webhook","enabled":true,"deliver3dsCode":true}'
```

```json
{
  "configured": true,
  "url": "https://yourapp.example/stablemesh/webhook",
  "enabled": true,
  "deliver3dsCode": true,
  "secretPrefix": "whsec_AbCd12",
  "secret": "whsec_AbCd12...full...XyZ"
}
```

{% hint style="warning" %}
The full **`secret`** is returned only once — at creation and on rotation. Store it; you'll need it
to verify signatures. Later `GET`s show only `secretPrefix`.
{% endhint %}

| Field | Description |
|-------|-------------|
| `url` | Your HTTPS endpoint (required) |
| `enabled` | Pause/resume deliveries without losing config |
| `deliver3dsCode` | If `true`, 3DS events include the one-time code (see [3DS](#card-3ds-requested)) |

---

## Delivery, retries & idempotency

- Each event is `POST`ed as JSON. Respond with any **2xx** to acknowledge.
- Non-2xx or a timeout is retried with exponential backoff — up to **6 attempts** (~1m, 5m, 30m, 2h, 5h, 10h).
- Retries reuse the same `id`. **Deduplicate on the event `id`** — treat delivery as at-least-once.
- Requests carry these headers:

| Header | Value |
|--------|-------|
| `StableMesh-Signature` | `t=<unix>,v1=<hex hmac>` — see below |
| `StableMesh-Event-Id` | the event `id` (for idempotency) |
| `StableMesh-Event-Type` | e.g. `transaction.updated` |

---

## Verifying signatures

Each request is signed with HMAC-SHA256 over `"<timestamp>.<raw request body>"` using your webhook
secret. Recompute it and compare in constant time; reject if the timestamp is too old.

```
StableMesh-Signature: t=1717243800,v1=4f1e9d…<64 hex chars>
```

```python
import hmac, hashlib, time

def verify(secret: str, header: str, raw_body: bytes, tolerance=300) -> bool:
    parts = dict(p.split("=", 1) for p in header.split(","))
    t, v1 = parts["t"], parts["v1"]
    if abs(time.time() - int(t)) > tolerance:
        return False
    expected = hmac.new(secret.encode(), f"{t}.".encode() + raw_body,
                        hashlib.sha256).hexdigest()
    return hmac.compare_digest(expected, v1)
```

---

## Event envelope

Every event has the same shape; `data` varies by `type`.

```json
{
  "id": "evt_9aF2b3c4...",
  "type": "transaction.updated",
  "created": 1717243800,
  "data": { ... }
}
```

| Event type | When |
|------------|------|
| `transaction.created` | A new card transaction is first seen (usually a `PENDING` authorization) |
| `transaction.updated` | A transaction changes state (`PENDING`→`CLEARED`/`DECLINED`/`CANCELLED`/`REFUND`) |
| `card.3ds.requested` | A 3DS / payment OTP challenge was raised on a card |
| `card.created` | A card was issued |
| `card.frozen` / `card.unfrozen` | A card was frozen / unfrozen |
| `card.cancelled` | A card was cancelled |

---

## Transaction events

`transaction.created` and `transaction.updated`:

```json
{
  "id": "evt_...", "type": "transaction.updated", "created": 1717243800,
  "data": {
    "transactionId": "txn_abc",
    "cardId": "2026060117803180277463",
    "status": "CLEARED",
    "previousStatus": "PENDING",
    "billAmount": 12.50,
    "billCurrency": "USD",
    "transactionAmount": 12.50,
    "transactionCurrency": "USD",
    "merchantName": "GOOGLE ADS",
    "merchantCountry": "US",
    "mccCategory": "advertising",
    "eventType": "SPEND",
    "settledAt": "2026-06-01T10:15:00"
  }
}
```

`status` values: `PENDING`, `CLEARED`, `DECLINED`, `CANCELLED`, `REFUND`. `previousStatus` is present
only on `transaction.updated`.

---

## card.3ds.requested

Raised when a card needs a 3DS / payment OTP. For a **headless** integration this lets you complete
the challenge without a human inbox.

```json
{
  "id": "evt_...", "type": "card.3ds.requested", "created": 1717243800,
  "data": {
    "type": "payment",
    "merchant": "GOOGLE ADS",
    "amount": "12.50",
    "currency": "USD",
    "otp": "123456",
    "cardId": "2026060117803180277463",
    "last4": "2222"
  }
}
```

- `type`: `payment` or `binding` (device/card activation).
- `otp`: the one-time code — **present only when your endpoint has `deliver3dsCode=true`**. Set it to
  `false` to receive the notification without the code.
- `cardId` / `last4`: each card has a dedicated inbox, so the OTP resolves to the exact card and the
  event carries its `cardId` and `last4`. (If a card has no dedicated inbox yet, `cardId` is set only
  when the account owns a single card.)

{% hint style="warning" %}
The OTP is a live secret. It is delivered over HTTPS and signed, and expires in ~10 minutes. Verify
the signature before trusting it, and never log it.
{% endhint %}

---

## Card lifecycle events

```json
{ "id": "evt_...", "type": "card.frozen", "created": 1717243800,
  "data": { "cardId": "2026060117803180277463" } }
```

`card.created` additionally carries `binId` and `status`.
