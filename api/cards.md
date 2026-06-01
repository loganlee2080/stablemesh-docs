# Cards

All card endpoints are `POST` under `/v1`, authenticated with the `X-API-KEY` header. Cards are issued on the **ether.fi** BIN and funded from your account's USDT balance.

---

## List available BINs

`POST /v1/card/binList`

Returns the card BINs enabled for your account.

**Request** `{}`

**Response**

```json
{
  "availableCardBinList": [
    {
      "id": "VISA-EFI-454924",
      "title": "Ether Card",
      "cardNetwork": "Visa",
      "enabled": true
    }
  ]
}
```

---

## Create a card

`POST /v1/card/create`

Issues a new card and funds it from your USDT balance. The requested `amount` is deducted from your USDT balance; the card is credited the net (after any deposit fee).

**Request**

```json
{
  "amount": 50,
  "currency": "USDT",
  "binId": "VISA-EFI-454924",
  "alias": "Marketing card"
}
```

| Field | Required | Description |
|-------|----------|-------------|
| `amount` | yes | Initial load, deducted from your USDT balance |
| `currency` | yes | `USDT` |
| `binId` | no | Defaults to the ether.fi BIN `VISA-EFI-454924` |
| `alias` | no | A label for the card |

**Response** — the created card object

```json
{
  "cardId": "card_xxx",
  "binId": "VISA-EFI-454924",
  "cardStatus": "ACTIVE",
  "maskedCardNo": "**** **** **** 2222",
  "cardBrand": "Visa"
}
```

**Example**

```bash
curl -X POST https://api.stablemesh.io/v1/card/create \
  -H "X-API-KEY: $SMK_KEY" -H "Content-Type: application/json" \
  -d '{"amount":50,"currency":"USDT","alias":"Marketing card"}'
```

Common errors: `3001 ASSET_NOT_ENOUGH` (USDT balance too low), `4009 CARD_OUT_OF_INVENTORY` (no card inventory available — contact support), `8001 DEPOSIT_AMOUNT_TOO_SMALL`.

---

## List cards

`POST /v1/card/list`

**Request**

```json
{ "status": "ACTIVE" }
```

`status` is optional; omit to return all cards. Values: `ACTIVE`, `FROZEN`, etc.

**Response**

```json
{
  "cardList": [
    {
      "cardId": "card_xxx",
      "binId": "VISA-EFI-454924",
      "cardStatus": "ACTIVE",
      "maskedCardNo": "**** **** **** 2222",
      "availableBalance": 49.0,
      "frozenAmount": 0,
      "currency": "USD"
    }
  ]
}
```

---

## Card detail

`POST /v1/card/detail` — full detail for one card.

**Request** `{ "cardId": "card_xxx" }` → returns the card detail object.

---

## Card balance

`POST /v1/card/asset` — the card's balance.

**Request** `{ "cardId": "card_xxx" }`

**Response**

```json
{ "availableAmount": "49", "frozenAmount": "0", "currency": "USD" }
```

---

## Top up a card

`POST /v1/card/deposit`

Adds funds to an existing card by **deducting from your USDT balance**.

**Request**

```json
{ "cardId": "card_xxx", "amount": 100, "currency": "USDT" }
```

| Field | Required | Description |
|-------|----------|-------------|
| `cardId` | yes | Card to fund |
| `amount` | yes | Positive amount, deducted from your USDT balance |
| `currency` | no | Defaults to `USDT` |

**Response** `{ "code": 0, "message": "ok" }`

Common errors: `3001 ASSET_NOT_ENOUGH`, `4001 CARD_NOT_FOUND`.

---

## Reveal sensitive info

`POST /v1/card/sensitiveInfo` — returns PAN, CVV, and expiry.

**Request** `{ "cardId": "card_xxx" }`

{% hint style="warning" %}
Sensitive card data is PCI-scoped. Only request it when you need to display or use the full card number, and never log or persist it.
{% endhint %}

---

## Freeze / Unfreeze / Cancel

`POST /v1/card/freeze` · `POST /v1/card/unfreeze` · `POST /v1/card/cancel`

Each takes the same body and returns the standard success envelope.

**Request** `{ "cardId": "card_xxx" }`

**Response** `{ "code": 0, "message": "ok" }`

```bash
curl -X POST https://api.stablemesh.io/v1/card/freeze \
  -H "X-API-KEY: $SMK_KEY" -H "Content-Type: application/json" \
  -d '{"cardId":"card_xxx"}'
```
