# Authentication

The Open API uses a single **secret API key** sent in the `X-API-KEY` header. There is no request signing, no timestamp, and no nonce — just your key over HTTPS.

---

## Your API key

A key looks like:

```
smk_AbCd1234EfGh5678IjKl...
```

- Keys are issued by the Stable Mesh team during onboarding.
- The **full secret is shown only once**, at creation. Store it in your secret manager — it cannot be retrieved again.
- Stable Mesh stores only a hash of your key, never the key itself.

---

## Sending the key

Add the `X-API-KEY` header to every request:

```bash
curl -X POST https://api.stablemesh.io/v1/card/list \
  -H "X-API-KEY: smk_AbCd1234EfGh5678IjKl..." \
  -H "Content-Type: application/json" \
  -d '{}'
```

---

## Errors

| HTTP | Meaning |
|------|---------|
| `401 Unauthorized` | Missing, unknown, or revoked API key. Response: `{"code":11002,"message":"Invalid or revoked API key"}` |
| `400 Bad Request` | Authenticated, but the request was invalid (see [Errors](errors.md)) |

---

## Rotating & revoking keys

You can hold more than one active key (useful for zero-downtime rotation). To rotate:

1. Ask Stable Mesh to issue a new key.
2. Deploy it to your systems.
3. Ask Stable Mesh to revoke the old key.

{% hint style="warning" %}
Treat your API key like a password. Anyone with it can create cards and move funds from your USDT balance. Never embed it in client-side code or commit it to source control. If a key is exposed, request revocation immediately.
{% endhint %}
