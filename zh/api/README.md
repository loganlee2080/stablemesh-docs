# 开放 API

Stable Mesh **开放 API** 让客户以编程方式发行和管理卡片。这是一套服务器到服务器的 REST API，使用单一密钥 **API Key** 进行认证（Stripe 风格）。

---

## 基础地址

| 环境 | Base URL |
|------|----------|
| 生产环境 | `https://api.stablemesh.io` |
| 沙盒 / 开发 | `https://dev-open.stablemesh.io` |

所有接口均在 `/v1` 前缀下，例如 `https://api.stablemesh.io/v1/card/create`。

---

## 运作方式

- **一个账户，一个余额。** 你的账户持有单一 **USDT 余额**。通过链上充值 USDT 来充值。
- **卡片从余额扣款。** 创建卡片或为卡片充值都会从你的 **USDT 余额扣款**。卡片消费额度与卡片余额实时同步，卡片永远不会超出你充值的金额。
- **卡片库存。** 卡片在 Visa BIN 上发行，库存由 Stable Mesh 预先准备；`card/create` 会从库存中为你分配卡片。
- **幂等重试。** 在 `card/create`、`card/deposit`、`card/freeze`、`card/unfreeze`、`card/cancel` 等会动账的接口上携带 `Idempotency-Key` 请求头（建议用 UUID），重试同一请求会原样返回首次的响应，不会重复发卡或重复扣款。详见英文文档 **Idempotency** 章节。

---

## 认证

每个请求都需在 `X-API-KEY` 请求头中携带你的密钥：

```bash
curl -X POST https://api.stablemesh.io/v1/card/list \
  -H "X-API-KEY: smk_AbCd1234..." \
  -H "Content-Type: application/json" \
  -d '{}'
```

{% hint style="info" %}
完整的接口字段、请求/响应示例与错误码以英文版为准，请参阅英文文档的 **Open API** 章节（Authentication / Wallet & Funding / Cards / Transactions / Errors）。如需开通账户与 API Key，请联系你的客户经理。
{% endhint %}
