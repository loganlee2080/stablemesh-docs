# Errors

The API uses standard HTTP status codes plus an application `code` in the JSON body.

- **`200 OK`** — success. Action endpoints return `{ "code": 0, "message": "ok" }`.
- **`400 Bad Request`** — a business/validation error. Body: `{ "code": <non-zero>, "message": "..." }`.
- **`401 Unauthorized`** — missing, unknown, or revoked API key.

Always check the `code` field even on a `200` response.

---

## Common error codes

| `code` | Name | Meaning |
|--------|------|---------|
| `0` | SUCCESS | OK |
| `400` | ILLEGAL_PARAMETER | A required field is missing or invalid |
| `3000` | ASSET_NOT_FOUND | No balance for the requested currency |
| `3001` | ASSET_NOT_ENOUGH | USDT balance too low for this action |
| `4001` | CARD_NOT_FOUND | No such card for your account |
| `4004` | BIN_NOT_FOUND | Unknown card BIN |
| `4005` | BIN_DISABLED | The card BIN is disabled |
| `4007` | REJECTED_HAVING_PENDING_ORDERS | A conflicting operation is pending on this card |
| `4008` | CARD_IS_DELETING | Card is being cancelled |
| `4009` | CARD_OUT_OF_INVENTORY | No card inventory available — contact support |
| `8001` | DEPOSIT_AMOUNT_TOO_SMALL | Amount below the minimum (or too small to cover the deposit fee) |
| `11002` | AUTH_FAILED | Invalid or revoked API key (HTTP 401) |
| `9999` | SYSTEM_ERROR | Unexpected server error — retry later |

---

## Example error response

```json
{
  "code": 3001,
  "message": "Your balance is too low for this action. Please add funds to your wallet and try again."
}
```

{% hint style="info" %}
On a `3001` (insufficient balance), top up your USDT balance — see [Wallet & Funding](wallet.md) — and retry.
{% endhint %}
