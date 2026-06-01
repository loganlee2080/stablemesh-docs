# Transactions

`POST /v1/bill/list`

Returns a paginated list of card transactions, newest first. Filter by a single card, by BIN, or across all your cards.

---

## Request

```json
{
  "cardId": "card_xxx",
  "pageNumber": 1,
  "pageSize": 20
}
```

| Field | Required | Description |
|-------|----------|-------------|
| `cardId` | no | Restrict to one card. Omit to query across your account. |
| `binId` | no | Restrict to a BIN (used when `cardId` is omitted). |
| `pageNumber` | no | 1-based page number. Defaults to the first page. |
| `pageSize` | no | Page size. Defaults to `20`. |

If you pass a `pageNumber` beyond the last page, the API returns the last available page rather than an error.

---

## Response

```json
{
  "transactions": [
    {
      "transactionTime": "2026-06-01T10:15:00Z",
      "merchant": "GOOGLE ADS",
      "amount": "-12.50",
      "currency": "USD",
      "status": "COMPLETED"
    }
  ],
  "last": false,
  "totalPages": 5,
  "totalElements": 92
}
```

| Field | Description |
|-------|-------------|
| `transactions` | The page of transactions |
| `last` | `true` if this is the final page |
| `totalPages` | Total number of pages |
| `totalElements` | Total transactions matching the query |

---

## Transaction statuses

| Status | Meaning |
|--------|---------|
| `COMPLETED` | Settled and funds deducted |
| `PENDING` | Authorised; funds held (frozen) but not yet settled |
| `FAILED` | Declined or failed |
| `REVERTED` | Reversed and funds returned |

---

## Example

```bash
curl -X POST https://api.stablemesh.io/v1/bill/list \
  -H "X-API-KEY: $SMK_KEY" -H "Content-Type: application/json" \
  -d '{"cardId":"card_xxx","pageNumber":1,"pageSize":20}'
```
