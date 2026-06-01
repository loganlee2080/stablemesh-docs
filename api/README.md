# B2B Open API

The Stable Mesh **Open API** lets business clients issue and manage cards programmatically. It is a server-to-server REST API authenticated with a single secret **API key** — the same Stripe-style model you already know.

This reference covers the **v1** API for **ether.fi** cards.

---

## Base URLs

| Environment | Base URL |
|-------------|----------|
| Production | `https://api.stablemesh.io` |
| Sandbox / Dev | `https://dev-open.stablemesh.io` |

All endpoints below are relative to a base URL and live under the `/v1` prefix, e.g. `https://api.stablemesh.io/v1/card/create`.

---

## How it works

- **One account, one balance.** Your account holds a single **USDT balance**. You top it up by depositing USDT on-chain (see [Wallet & Funding](wallet.md)).
- **Cards draw from that balance.** Creating a card or topping one up **deducts from your USDT balance**. The card's spending limit is kept in lock-step with its balance, so a card can never spend more than you funded.
- **ether.fi cards.** Cards are issued on the ether.fi BIN. Inventory is provisioned by Stable Mesh ahead of time; `card/create` assigns you a card from that inventory.

---

## Conventions

- Every request is `POST` with a JSON body and a `Content-Type: application/json` header.
- Authentication is the `X-API-KEY` header on every request — see [Authentication](authentication.md).
- All monetary amounts are decimal strings/numbers. Card balances are denominated in **USD**; your wallet balance is **USDT**.
- Responses are JSON. Action endpoints return an envelope `{ "code": 0, "message": "ok" }` on success; `code` is non-zero on error (see [Errors](errors.md)).

---

## Endpoints at a glance

| Area | Endpoint | Purpose |
|------|----------|---------|
| [Wallet](wallet.md) | `POST /v1/wallet/assetList` | List your crypto balances |
| [Wallet](wallet.md) | `POST /v1/wallet/getAddress` | Get your deposit address |
| [Cards](cards.md) | `POST /v1/card/binList` | List available card BINs |
| [Cards](cards.md) | `POST /v1/card/create` | Issue a card |
| [Cards](cards.md) | `POST /v1/card/list` | List your cards |
| [Cards](cards.md) | `POST /v1/card/detail` | Card details |
| [Cards](cards.md) | `POST /v1/card/asset` | Card balance |
| [Cards](cards.md) | `POST /v1/card/deposit` | Top up a card |
| [Cards](cards.md) | `POST /v1/card/sensitiveInfo` | Reveal PAN / CVV / expiry |
| [Cards](cards.md) | `POST /v1/card/freeze` · `unfreeze` · `cancel` | Manage card state |
| [Transactions](transactions.md) | `POST /v1/bill/list` | List card transactions |

{% hint style="info" %}
Need credentials? API keys are issued by the Stable Mesh team during onboarding. Contact your account manager to provision an account and your first key.
{% endhint %}
