# Idempotency

Network calls fail. When a request times out you often can't tell whether it succeeded — retrying a card create might make **two** cards, retrying a deposit might **double-debit** your USDT. Idempotency keys make retries safe: a retried request returns the **original** response instead of executing again.

## How it works

Send an `Idempotency-Key` header with a unique value (a UUID is ideal) for each logical operation you want to be retry-safe:

```http
POST /v1/card/create HTTP/1.1
Host: api.stablemesh.io
X-API-KEY: smk_live_…
Idempotency-Key: 1f0c8a2e-4b9d-4f3a-8b21-7c6e5d4a3b21
Content-Type: application/json

{ "amount": 50, "currency": "USDT" }
```

- The **first** request with a given key executes normally and its response is stored.
- Any **retry** with the **same key** replays the stored response verbatim — no second card, no second debit. The replay carries an `Idempotent-Replayed: true` response header.
- Only **successful (2xx)** responses are stored. If the first attempt failed (a 4xx/5xx), no result is kept, so you can safely retry the operation.
- Keys are scoped to **your account** and retained for **24 hours**.

The key you sent is echoed back in the `Idempotency-Key` response header.

## Where it applies

Idempotency keys are honored on the mutating card endpoints, where a double-execution would create a card or move funds:

| Endpoint | Without a key, a retry could… |
|---|---|
| `POST /v1/card/create` | create a duplicate card and debit USDT twice |
| `POST /v1/card/deposit` | top the card up twice |
| `POST /v1/card/freeze` / `unfreeze` / `cancel` | re-issue a redundant lifecycle change |

Read-only endpoints (listing cards, balances, transactions) don't need a key — they're already safe to repeat.

## Edge cases

| Situation | Result |
|---|---|
| Same key, **identical** request | Replays the original response (`Idempotent-Replayed: true`). |
| Same key, **different** request body | `409` with code `4211` (`IDEMPOTENCY_KEY_MISMATCH`) — reuse a key only for the exact same call. |
| Same key, first request **still in flight** | `409` with code `4212` (`IDEMPOTENCY_KEY_IN_PROGRESS`) — back off briefly and retry. |
| Key reused after **24 hours** | Treated as a fresh request. |

## Best practices

- Generate **one key per logical operation** (e.g. per "fund this card" action), and reuse that same key for every retry of it.
- Use a **random UUID** — never a predictable or shared value.
- Keep retrying with the same key (with backoff) until you get a definitive response. A `409 IN_PROGRESS` means the original is still running; a 2xx (possibly replayed) means it's done.

> By default the key is **optional** — sent it and it's honored, omit it and the request behaves normally. Enterprise tenants can ask us to make it **required** on mutations (a missing key then returns `400` code `4210`).
