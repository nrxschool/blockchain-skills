---
name: horizon-legacy
description: Horizon REST API for classic Stellar operations — accounts, transactions, effects, streaming. Use when maintaining legacy integrations. Do NOT use for new projects — prefer Stellar RPC.
metadata:
  author: nearx
  version: '1.0.0'
  source: 'Stellar Docs + stellarskills'
---

# Horizon (Legacy REST API)

> Horizon is the legacy REST API for Stellar. It is **deprecated** for new integrations. Migrate to Stellar RPC when possible.

## ⚠️ Deprecation Notice
Horizon is being deprecated in favor of Stellar RPC. For new projects, use Stellar RPC instead.
- Migration guide: https://developers.stellar.org/docs/data/apis/migrate-from-horizon-to-rpc

## Endpoints

| Network | URL |
|---------|-----|
| **Mainnet** | `https://horizon.stellar.org` |
| **Testnet** | `https://horizon-testnet.stellar.org` |

## JavaScript SDK

```javascript
import { Horizon } from "@stellar/stellar-sdk";

const server = new Horizon.Server("https://horizon-testnet.stellar.org");

// Load account
const account = await server.loadAccount(publicKey);
console.log("Balances:", account.balances);

// Get transactions
const txs = await server.transactions()
  .forAccount(publicKey)
  .limit(10)
  .order("desc")
  .call();

// Get payments
const payments = await server.payments()
  .forAccount(publicKey)
  .limit(20)
  .call();
```

## Streaming (Server-Sent Events)

```javascript
// Stream payments in real-time
server.payments()
  .forAccount(publicKey)
  .cursor("now")
  .stream({
    onmessage: (payment) => {
      console.log("New payment:", payment);
    },
    onerror: (error) => {
      console.error("Stream error:", error);
    },
  });
```

## Key Endpoints

| Endpoint | Description |
|----------|------------|
| `GET /accounts/{id}` | Account details, balances, signers |
| `GET /accounts/{id}/transactions` | Account transaction history |
| `GET /accounts/{id}/payments` | Payment history |
| `GET /accounts/{id}/effects` | Effects on account |
| `GET /accounts/{id}/offers` | Open DEX offers |
| `GET /transactions/{hash}` | Transaction details |
| `GET /ledgers` | Ledger history |
| `GET /order_book` | DEX order book |
| `POST /transactions` | Submit transaction |

## When to Still Use Horizon
- Querying account balances and history
- Streaming real-time events
- DEX order book queries
- Legacy system integrations

## Official Documentation
- Horizon: https://developers.stellar.org/docs/data/apis/horizon
- Migrate to RPC: https://developers.stellar.org/docs/data/apis/migrate-from-horizon-to-rpc
