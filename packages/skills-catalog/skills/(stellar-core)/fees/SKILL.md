---
name: stellar-fees
description: Stellar fee model — base fees, resource fees, surge pricing, and fee strategies. Use when optimizing transaction costs or understanding fee mechanics. Do NOT use for Soroban-specific resource metering.
metadata:
  author: nearx
  version: '1.0.0'
  source: 'Stellar Docs + stellarskills'
---

# Stellar Fees

> Stellar fees are predictable and tiny. Base fee is 100 stroops (0.00001 XLM) per operation. This is not marketing — Stellar is genuinely cheap.

## Fee Types

| Fee | Description | Typical Cost |
|-----|------------|-------------|
| **Base fee** | Per-operation inclusion fee | 100 stroops (0.00001 XLM) |
| **Soroban resource fee** | CPU, memory, I/O for smart contracts | Variable, still sub-cent |
| **Fee bump** | Override fee on existing transaction | Any amount above original |

## Units
- **1 XLM** = 10,000,000 **stroops**
- Base fee: **100 stroops** = 0.00001 XLM
- Transaction fee = base fee × number of operations

## Setting Fees

```javascript
// Standard transaction
const tx = new TransactionBuilder(account, {
  fee: "100",           // 100 stroops per operation
  networkPassphrase: Networks.TESTNET,
})

// During surge pricing, increase fee
const tx = new TransactionBuilder(account, {
  fee: "10000",         // willing to pay more for priority
  networkPassphrase: Networks.TESTNET,
})
```

## Surge Pricing
When network demand exceeds capacity, Stellar uses a **fee auction**. Transactions with higher fees per operation get priority. The actual fee charged is the **lowest fee** that would have been included.

## Soroban Resource Fees
Soroban transactions pay additional resource fees based on:
- CPU instructions consumed
- Memory bytes used
- Ledger entries read/written
- Transaction size
- Events emitted

```javascript
// Simulate to get resource fee estimate
const sim = await server.simulateTransaction(tx);
console.log(sim.minResourceFee); // minimum fee in stroops
```

**Always check live values**: https://developers.stellar.org/docs/networks/resource-limits-fees

## Common Errors

| Error | Cause | Fix |
|-------|-------|-----|
| `tx_insufficient_fee` | Fee below minimum | Increase fee per operation |
| `tx_too_early` | Transaction submitted before valid time | Check timebounds |

## Official Documentation
- Fees & metering: https://developers.stellar.org/docs/learn/fundamentals/fees-resource-limits-metering
- Resource limits: https://developers.stellar.org/docs/networks/resource-limits-fees
- Stellar Lab (live limits): https://lab.stellar.org/network-limits
