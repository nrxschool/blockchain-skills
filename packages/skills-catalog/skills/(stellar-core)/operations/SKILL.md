---
name: stellar-operations
description: All Stellar transaction operations — Payment, CreateAccount, ManageOffers, PathPayment, and more. Use when building or submitting transactions. Do NOT use for Soroban contract invocations.
metadata:
  author: nearx
  version: '1.0.0'
  source: 'Stellar Docs + stellarskills'
---

# Stellar Operations

> Stellar transactions contain typed operations. A single transaction can hold up to 100 operations, and they execute atomically — all succeed or all fail.

## Transaction Structure

```javascript
import { TransactionBuilder, Operation, Networks, Keypair, Horizon } from "@stellar/stellar-sdk";

const server = new Horizon.Server("https://horizon-testnet.stellar.org");
const account = await server.loadAccount(sourcePublicKey);

const tx = new TransactionBuilder(account, {
  fee: "100",          // base fee per operation in stroops
  networkPassphrase: Networks.TESTNET,
})
  .addOperation(Operation.payment({ /* ... */ }))
  .addOperation(Operation.payment({ /* ... */ })) // batch multiple ops
  .setTimeout(30)      // seconds until tx expires
  .build();

tx.sign(keypair);
const result = await server.submitTransaction(tx);
```

## Core Operations

### Payment
```javascript
Operation.payment({
  destination: "GDEST...",
  asset: Asset.native(),    // XLM
  amount: "100",
})
```

### Create Account
```javascript
Operation.createAccount({
  destination: "GNEW...",
  startingBalance: "2",     // minimum ~1 XLM
})
```

### Path Payment (Strict Send)
Convert assets automatically using the built-in DEX:
```javascript
Operation.pathPaymentStrictSend({
  sendAsset: Asset.native(),
  sendAmount: "10",
  destination: "GDEST...",
  destAsset: usdcAsset,
  destMin: "9.5",        // minimum to receive
  path: [],              // auto-find path
})
```

### Path Payment (Strict Receive)
```javascript
Operation.pathPaymentStrictReceive({
  sendAsset: Asset.native(),
  sendMax: "11",
  destination: "GDEST...",
  destAsset: usdcAsset,
  destAmount: "10",       // exact amount to deliver
  path: [],
})
```

### Manage Sell Offer (DEX)
```javascript
Operation.manageSellOffer({
  selling: myAsset,
  buying: Asset.native(),
  amount: "100",
  price: "0.5",           // 0.5 XLM per token
  offerId: "0",           // 0 = new offer
})
```

### Manage Buy Offer (DEX)
```javascript
Operation.manageBuyOffer({
  selling: Asset.native(),
  buying: myAsset,
  buyAmount: "100",
  price: "2",
  offerId: "0",
})
```

### Change Trust
```javascript
Operation.changeTrust({
  asset: new Asset("USDC", issuerPublicKey),
  limit: "1000000",       // optional max
})
```

### Set Options
```javascript
Operation.setOptions({
  homeDomain: "nearx.com",
  inflationDest: "GINFLATION...",
  signer: { ed25519PublicKey: signerKey, weight: 1 },
})
```

### Manage Data
```javascript
Operation.manageData({
  name: "config_key",
  value: Buffer.from("config_value"),  // null to delete
})
```

## Fee Bumps

Wrap an existing transaction with a higher fee:
```javascript
const feeBumpTx = TransactionBuilder.buildFeeBumpTransaction(
  feeSourceKeypair,
  "500",               // new higher fee
  innerTransaction,
  Networks.TESTNET,
);
feeBumpTx.sign(feeSourceKeypair);
```

## Common Errors

| Error | Cause | Fix |
|-------|-------|-----|
| `tx_bad_seq` | Wrong sequence number | Reload account from network |
| `tx_too_late` | Transaction expired | Increase setTimeout or rebuild |
| `op_underfunded` | Not enough balance | Check balance before operation |
| `op_low_reserve` | Would drop below minimum balance | Ensure enough XLM for reserves |
| `tx_bad_auth` | Wrong/missing signatures | Sign with correct keypair for network |

## Official Documentation
- Operations list: https://developers.stellar.org/docs/learn/fundamentals/transactions/list-of-operations
- Transactions: https://developers.stellar.org/docs/learn/fundamentals/transactions
- Fee bumps: https://developers.stellar.org/docs/learn/encyclopedia/transactions-specialized/fee-bump-transactions
