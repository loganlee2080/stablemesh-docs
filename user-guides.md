# Stable Mesh User Guide

Stable Mesh is a virtual credit card (VCC) platform that lets you fund Mastercard-powered cards using on-chain stablecoins (USDT, USDC) and spend them anywhere Mastercard is accepted — online and in-store.

---

## Table of Contents

1. [Overview](#overview)
2. [Dashboard](#dashboard)
3. [Deposit Crypto](#deposit-crypto)
4. [Cards](#cards)
   - [View All Cards](#view-all-cards)
   - [Card Detail](#card-detail)
   - [View Sensitive Info](#view-sensitive-info)
   - [Freeze a Card](#freeze-a-card)
   - [Unfreeze a Card](#unfreeze-a-card)
   - [Top Up a Card](#top-up-a-card)
   - [View Transactions](#view-transactions)
5. [Create a Card](#create-a-card)
6. [Invite & Earn](#invite--earn)

---

## Overview

After signing in, you land on the **Home** dashboard. The left sidebar gives you three sections:

| Section | What it does |
|---------|-------------|
| **Home** | Balance overview and recent transactions |
| **Cards** | Manage all your virtual cards |
| **Profile** | Account settings |

---

## Dashboard

The dashboard gives you a real-time snapshot of your account:

- **Total Balance** — the combined value of your crypto holdings and all card balances.
- **Crypto Assets** — your on-chain wallet balances (USDT, USDC).
- **USD Cards** — a summary of each card with its available balance.
- **Recent Transactions** — the latest spending activity across all cards.

---

## Deposit Crypto

Before you can top up a card you need crypto in your Stable Mesh wallet.

1. On the **Home** page, click the **+ Deposit** button.
2. Select the network from the dropdown next to Deposit (e.g. **SOL**).
3. Follow the on-screen instructions to copy your deposit address and send USDT or USDC from your external wallet.

> Your balance updates automatically once the transaction is confirmed on-chain.

---

## Cards

### View All Cards

Click **Cards** in the left sidebar to see every card you own. The table shows:

- **Alias** — the name you gave the card
- **Card Number** — last 4 digits
- **Available Balance** — spendable USD on the card
- **Status** — `ACTIVE` (green) or `FROZEN` (orange)

---

### Card Detail

Click any card row to open the detail panel on the right. It shows:

- The card image with masked card number
- Three action buttons: **View Info**, **Freeze / Unfreeze**, **Delete**
- Available balance with a **Recharge** button
- Full transaction history (消费记录) scrollable below

---

### View Sensitive Info

To see your full card number, expiry date, and CVV:

1. Open the card detail panel (click a card row).
2. Click the **eye icon** (查看详情).
3. The card flips to reveal **Card Number**, **Expire**, and **CVV**.
4. Click the eye icon again (**隐藏详情**) to hide the details.

> Keep your CVV private — never share it with anyone.

---

### Freeze a Card

Freezing a card instantly blocks all new transactions without deleting it. Use this if you suspect unauthorised use or want to temporarily pause spending.

1. Open the card detail panel.
2. Click the **snowflake icon** (冻结).
3. A confirmation dialog appears: **"确认冻结卡片?"**
4. Click **OK** to confirm. The card status changes to `FROZEN`.

---

### Unfreeze a Card

1. Open the detail panel of a **FROZEN** card.
2. The snowflake icon is now highlighted green and labelled **解冻**.
3. Click **解冻**.
4. Confirm the dialog: **"确认解冻卡片?"** → click **OK**.
5. The card returns to `ACTIVE` status and can accept transactions again.

---

### Top Up a Card

Move funds from your crypto wallet onto a specific card.

1. Open the card detail panel.
2. Click the green **Recharge** button.
3. In the modal:
   - **Card** — pre-filled with the selected card name.
   - **Currency** — select `USDT` or `USDC` from the dropdown. Your available balance is shown next to each option.
   - **Amount** — enter the USD amount you want to load.
4. Click **Recharge** to confirm.

> The amount is deducted from your crypto wallet and added to the card balance immediately.

---

### View Transactions

Each card keeps a full log of its spending history.

1. Open the card detail panel.
2. Scroll down to the **消费记录** (Transaction History) section.
3. Each entry shows:
   - **Merchant name**
   - **Date and time**
   - **Amount** (in USD)
   - **Status**: `COMPLETED`, `FAILED`, or `REVERTED`

---

## Create a Card

1. From the **Cards** page, click **Create Card** (top right).
2. The **Start Card Application** screen opens showing the available card type — **Revo Card** (global Mastercard, online and in-store, lowest fees).
3. Click **Next** and follow the steps to configure and activate your new card.

> Newly created cards start with a $0 balance. Top them up after creation.

---

## Invite & Earn

Click **Invite & Earn** in the top navigation bar to access your personal referral link. Share it with friends — you both earn rewards when they sign up and use the platform.

---

*For additional support, contact the Stable Mesh team through the official channels.*
