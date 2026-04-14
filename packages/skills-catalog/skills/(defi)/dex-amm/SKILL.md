---
name: dex-amm
description: Stellar built-in DEX and AMM — order books, manage offers, path payments, liquidity pools. Use when trading on Stellar or building DEX integrations. Do NOT use for external DeFi protocols or Soroban-based DEXes.
metadata:
  author: nearx
  version: '1.0.0'
  source: 'Stellar Docs + stellarskills'
---

# DEX & AMM

> Stellar has a **protocol-native** DEX with both order book and AMM. No smart contract needed — trading is built into the ledger.

## Key Concepts

- **Order Book DEX** — Native limit orders, managed via operations
- **AMM (Liquidity Pools)** — Constant-product pools, built in since Protocol 18
- **Path Payments** — Auto-route through the DEX for best price
- **No Uniswap needed** — This is all protocol-level, not a smart contract

## Order Book Trading

### Place a Sell Offer
```javascript
import { Operation, Asset } from "@stellar/stellar-sdk";

// Sell 100 NRXTOKEN at 0.5 XLM each
const offer = Operation.manageSellOffer({
  selling: new Asset("NRXTOKEN", issuerPublicKey),
  buying: Asset.native(),
  amount: "100",
  price: "0.5",     // price in terms of buying asset
  offerId: "0",     // 0 = new offer
});
```

### Place a Buy Offer
```javascript
const offer = Operation.manageBuyOffer({
  selling: Asset.native(),
  buying: new Asset("NRXTOKEN", issuerPublicKey),
  buyAmount: "100",
  price: "2.0",     // max price to pay
  offerId: "0",
});
```

### Cancel an Offer
```javascript
Operation.manageSellOffer({
  selling: myAsset,
  buying: Asset.native(),
  amount: "0",       // 0 = cancel
  price: "1",
  offerId: existingOfferId,
});
```

### Query Order Book
```javascript
const orderBook = await server.orderbook(
  new Asset("USDC", usdcIssuer),
  Asset.native()
).call();

console.log("Bids:", orderBook.bids);
console.log("Asks:", orderBook.asks);
```

## Path Payments

Auto-route through the DEX for best conversion:

```javascript
// Send XLM, recipient gets USDC (best route auto-found)
Operation.pathPaymentStrictReceive({
  sendAsset: Asset.native(),
  sendMax: "100",             // max XLM to spend
  destination: recipientKey,
  destAsset: usdcAsset,
  destAmount: "50",           // exact USDC to deliver
  path: [],                   // empty = auto-find
});
```

### Find Payment Paths
```javascript
const paths = await server.strictReceivePaths(
  [Asset.native()],           // source assets
  usdcAsset,                  // destination asset
  "100"                       // destination amount
).call();

paths.records.forEach((path) => {
  console.log(`Send ${path.source_amount} XLM → Receive ${path.destination_amount} USDC`);
  console.log("Path:", path.path);
});
```

## AMM Liquidity Pools

### Deposit Liquidity
```javascript
// 1. Create trustline for LP share
const poolId = getLiquidityPoolId(
  "constant_product",
  { assetA: Asset.native(), assetB: usdcAsset, fee: 30 }
);

Operation.changeTrust({
  asset: new LiquidityPoolAsset(Asset.native(), usdcAsset, 30),
});

// 2. Deposit
Operation.liquidityPoolDeposit({
  liquidityPoolId: poolId,
  maxAmountA: "1000",    // max XLM
  maxAmountB: "500",     // max USDC
  minPrice: { n: 1, d: 2 },
  maxPrice: { n: 2, d: 1 },
});
```

### Withdraw Liquidity
```javascript
Operation.liquidityPoolWithdraw({
  liquidityPoolId: poolId,
  amount: "100",          // LP shares to burn
  minAmountA: "400",
  minAmountB: "200",
});
```

## Soroban-Based DEXes

For more complex trading logic (limit orders with custom conditions, etc.), see Soroban-based protocols:
- **Soroswap**: https://github.com/carstenjacobsen/ai-soroswap-integration
- **DeFindex**: https://github.com/carstenjacobsen/ai-defindex-integration

## Common Errors

| Error | Cause | Fix |
|-------|-------|-----|
| `MANAGE_OFFER_UNDERFUNDED` | Not enough of selling asset | Check balance |
| `MANAGE_OFFER_CROSS_SELF` | Would trade with yourself | Check existing offers |
| `MANAGE_OFFER_NOT_FOUND` | Offer ID doesn't exist | Verify offerId |
| `PATH_PAYMENT_NO_DESTINATION` | No viable path found | Try different assets or amounts |

## Official Documentation
- DEX overview: https://developers.stellar.org/docs/learn/fundamentals/stellar-data-structures/liquidity-on-stellar-sdex-liquidity-pools
- AMM pools: https://developers.stellar.org/docs/learn/encyclopedia/sdex/liquidity-on-stellar-sdex-liquidity-pools
