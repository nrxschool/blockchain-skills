---
name: seps-anchors
description: Stellar SEP standards and anchor ecosystem — SEP-6, SEP-10, SEP-24, SEP-31, stellar.toml, fiat on/off-ramps. Use when integrating with anchors or implementing SEP flows. Do NOT use for general Stellar operations or smart contracts.
metadata:
  author: nearx
  version: '1.0.0'
  source: 'Stellar Docs + stellarskills'
---

# SEPs & Anchors

> The anchor ecosystem enables fiat on/off-ramps via SEP standards. Anchors bridge traditional finance and the Stellar network.

## What is an Anchor?
An anchor is an entity that connects Stellar to traditional financial systems. They accept fiat deposits and issue corresponding tokens on Stellar (e.g., USDC, local currency tokens).

## Key SEP Standards

| SEP | Purpose | Type |
|-----|---------|------|
| **SEP-1** | `stellar.toml` discovery file | Configuration |
| **SEP-6** | Programmatic deposit/withdrawal | API-based |
| **SEP-10** | Web authentication | Authentication |
| **SEP-24** | Interactive deposit/withdrawal | Web-based (iframe) |
| **SEP-31** | Cross-border payments | Send-only |
| **SEP-38** | Quote API (exchange rates) | Pricing |

## SEP-1: stellar.toml

Every anchor publishes a `stellar.toml` file at `https://domain.com/.well-known/stellar.toml`:

```toml
# stellar.toml
ACCOUNTS = ["GISSUER..."]
TRANSFER_SERVER = "https://api.anchor.com/sep6"
TRANSFER_SERVER_SEP0024 = "https://api.anchor.com/sep24"
WEB_AUTH_ENDPOINT = "https://api.anchor.com/auth"
SIGNING_KEY = "GSIGNER..."

[[CURRENCIES]]
code = "USDC"
issuer = "GA5ZSEJYB37JRC5AVCIA5MOP4RHTM335X2KGX3IHOJAPP5RE34K4KZVN"
display_decimals = 2
```

```javascript
// Fetch and parse stellar.toml
const toml = await fetch("https://anchor.com/.well-known/stellar.toml");
const tomlText = await toml.text();
```

## SEP-10: Authentication

Web authentication flow — client proves account ownership to an anchor:

```javascript
// 1. Request challenge
const challengeRes = await fetch(
  `${webAuthEndpoint}?account=${publicKey}`
);
const { transaction } = await challengeRes.json();

// 2. Sign challenge transaction
const tx = TransactionBuilder.fromXDR(transaction, networkPassphrase);
tx.sign(keypair);

// 3. Submit signed challenge
const tokenRes = await fetch(webAuthEndpoint, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ transaction: tx.toXDR() }),
});

const { token } = await tokenRes.json(); // JWT for subsequent requests
```

## SEP-24: Interactive Deposit/Withdrawal

SEP-24 opens a web page (or iframe) for KYC and payment details:

```javascript
// 1. Authenticate via SEP-10 first (get JWT token)

// 2. Initiate interactive deposit
const depositRes = await fetch(`${transferServer}/transactions/deposit/interactive`, {
  method: "POST",
  headers: {
    Authorization: `Bearer ${jwtToken}`,
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    asset_code: "USDC",
    account: publicKey,
  }),
});

const { url } = await depositRes.json();
// 3. Open URL in browser/iframe for user to complete KYC and deposit

// 4. Poll for transaction completion
const statusRes = await fetch(
  `${transferServer}/transaction?id=${transactionId}`,
  { headers: { Authorization: `Bearer ${jwtToken}` } }
);
```

## SEP-6: Programmatic Deposit/Withdrawal

SEP-6 is fully API-based (no interactive web page):

```javascript
const depositRes = await fetch(
  `${transferServer}/deposit?asset_code=USDC&account=${publicKey}&type=bank_transfer`,
  { headers: { Authorization: `Bearer ${jwtToken}` } }
);

const depositInfo = await depositRes.json();
// Returns bank details for the user to send fiat
```

## Anchor Starter Pack

A great reference for building SEP-compliant integrations:
https://github.com/ElliotFriend/regional-starter-pack

## Flow Summary: Deposit Fiat → Get Tokens

1. **Discover** — Fetch `stellar.toml` from anchor domain
2. **Authenticate** — SEP-10 web auth to get JWT
3. **Trustline** — Create trustline for the anchor's asset
4. **Deposit** — SEP-24 (interactive) or SEP-6 (API)
5. **Receive** — Anchor sends tokens to your Stellar account

## Official Documentation
- SEP standards: https://developers.stellar.org/docs/learn/fundamentals/stellar-ecosystem-proposals
- Anchor integration guide: https://developers.stellar.org/docs/learn/interactive
- SEP-24 spec: https://github.com/stellar/stellar-protocol/blob/master/ecosystem/sep-0024.md
