# Wallet & Funding

Your account holds a single **USDT balance**. You fund it by depositing USDT on-chain to the address returned by `getAddress`; cards then draw from this balance.

---

## Funding flow

1. Call `POST /v1/wallet/getAddress` to get your USDT deposit address.
2. Send **USDT (TRC20)** to that address.
3. Once the deposit confirms on-chain, your USDT balance is credited automatically.
4. Use the balance to [create](cards.md#create-a-card) and [top up](cards.md#top-up-a-card) cards.

{% hint style="info" %}
The supported funding rail is **USDT on Tron (TRC20)**. Send only USDT-TRC20 to your deposit address.
{% endhint %}

---

## List balances

`POST /v1/wallet/assetList`

Returns your crypto balances (USDT / USDC).

**Request**

```json
{}
```

**Response**

```json
{
  "assetList": [
    {
      "currency": "USDT",
      "availableAmount": "1500",
      "frozenAmount": "0",
      "assetType": "CRYPTO",
      "status": "ACTIVE"
    },
    {
      "currency": "USDC",
      "availableAmount": "0",
      "assetType": "CRYPTO",
      "active": false
    }
  ]
}
```

| Field | Description |
|-------|-------------|
| `currency` | `USDT` or `USDC` |
| `availableAmount` | Spendable balance |
| `frozenAmount` | Funds held / pending |

---

## Get deposit address

`POST /v1/wallet/getAddress`

**Request**

```json
{
  "currency": "USDT",
  "network": "TRX"
}
```

| Field | Required | Description |
|-------|----------|-------------|
| `currency` | yes | `USDT` |
| `network` | yes | `TRX` (Tron / TRC20) |

**Response**

```json
{
  "code": 0,
  "message": "ok",
  "address": "TXxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
  "network": "TRX"
}
```

**Example**

```bash
curl -X POST https://api.stablemesh.io/v1/wallet/getAddress \
  -H "X-API-KEY: $SMK_KEY" \
  -H "Content-Type: application/json" \
  -d '{"currency":"USDT","network":"TRX"}'
```

{% hint style="warning" %}
Your deposit address is assigned during onboarding. If `getAddress` returns an error stating no address is configured, contact your account manager.
{% endhint %}
