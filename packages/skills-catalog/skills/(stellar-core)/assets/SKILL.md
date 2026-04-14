---
name: stellar-assets
description: Stellar asset issuance, trustlines, custom tokens, and Stellar Asset Contract (SAC). Use when issuing tokens, managing trustlines, or bridging classic assets to Soroban. Do NOT use for Soroban-native tokens or general smart contract development.
metadata:
  author: nearx
  version: '1.0.0'
  source: 'Stellar Docs + stellarskills'
---

# Stellar Assets

> Stellar has native support for asset issuance. Any account can issue assets, and the protocol handles trustlines, authorization, and clawback natively.

## Asset Types

| Type | Description |
|------|------------|
| **Native (XLM)** | Built-in, no issuer, used for fees and reserves |
| **Credit (alphanumeric)** | Issued by any account, requires trustlines |
| **Liquidity Pool Shares** | Represent LP positions in the built-in AMM |

## Issuing an Asset

```javascript
import { Asset, Keypair, Networks, TransactionBuilder, Operation, Horizon } from "@stellar/stellar-sdk";

const server = new Horizon.Server("https://horizon-testnet.stellar.org");

// 1. Issuer and distributor accounts
const issuer = Keypair.fromSecret("ISSUER_SECRET");
const distributor = Keypair.fromSecret("DISTRIBUTOR_SECRET");

const myAsset = new Asset("NRXTOKEN", issuer.publicKey());

// 2. Distributor creates trustline to issuer's asset
const distributorAccount = await server.loadAccount(distributor.publicKey());
const trustTx = new TransactionBuilder(distributorAccount, {
  fee: "100",
  networkPassphrase: Networks.TESTNET,
})
  .addOperation(Operation.changeTrust({ asset: myAsset }))
  .setTimeout(30)
  .build();

trustTx.sign(distributor);
await server.submitTransaction(trustTx);

// 3. Issuer sends tokens to distributor
const issuerAccount = await server.loadAccount(issuer.publicKey());
const payTx = new TransactionBuilder(issuerAccount, {
  fee: "100",
  networkPassphrase: Networks.TESTNET,
})
  .addOperation(
    Operation.payment({
      destination: distributor.publicKey(),
      asset: myAsset,
      amount: "1000000",
    })
  )
  .setTimeout(30)
  .build();

payTx.sign(issuer);
await server.submitTransaction(payTx);
```

## Trustlines

**Critical rule**: An account MUST establish a trustline before it can receive a non-XLM asset. Never assume trustlines exist.

```javascript
// Check if trustline exists
const account = await server.loadAccount(userPublicKey);
const hasTrustline = account.balances.some(
  (b) => b.asset_code === "USDC" && b.asset_issuer === USDC_ISSUER
);
```

## USDC on Stellar (Circle)

Always verify current issuers at: https://developers.circle.com/stablecoins/usdc-contract-addresses

| Network | Issuer |
|---------|--------|
| **Mainnet** | `GA5ZSEJYB37JRC5AVCIA5MOP4RHTM335X2KGX3IHOJAPP5RE34K4KZVN` |
| **Testnet** | `GBBD47IF6LWK7P7MDEVSCWR7DPUWV3NY3DTQEVFL4NAT4AQH3ZLLFLA5` |

```javascript
const USDC_MAINNET = new Asset("USDC", "GA5ZSEJYB37JRC5AVCIA5MOP4RHTM335X2KGX3IHOJAPP5RE34K4KZVN");
const USDC_TESTNET = new Asset("USDC", "GBBD47IF6LWK7P7MDEVSCWR7DPUWV3NY3DTQEVFL4NAT4AQH3ZLLFLA5");
```

## Stellar Asset Contract (SAC)

SAC bridges classic Stellar assets into Soroban. Every classic asset has a corresponding contract address:

```javascript
import { Asset, Networks } from "@stellar/stellar-sdk";

// Get SAC contract ID for any asset
const xlmContractId = Asset.native().contractId(Networks.PUBLIC);
const usdcContractId = new Asset("USDC", USDC_ISSUER).contractId(Networks.PUBLIC);
```

```rust
// Use SAC in Soroban contracts
use soroban_sdk::token;

let token_client = token::Client::new(&env, &sac_contract_id);
token_client.transfer(&from, &to, &amount);
let balance = token_client.balance(&user);
```

## Authorization Flags

Issuers can control asset behavior:

```javascript
.addOperation(Operation.setOptions({
  setFlags:
    AuthRequiredFlag |     // Recipients need issuer approval
    AuthRevocableFlag |    // Issuer can freeze assets
    AuthClawbackEnabledFlag // Issuer can clawback tokens
}))
```

## Common Errors

| Error | Cause | Fix |
|-------|-------|-----|
| `CHANGE_TRUST_INVALID_LIMIT` | Invalid trustline limit | Use positive limit or omit for max |
| `PAYMENT_NO_TRUST` | Recipient has no trustline | Recipient must `changeTrust` first |
| `PAYMENT_NOT_AUTHORIZED` | Issuer requires authorization | Issuer must `allowTrust` the recipient |
| `PAYMENT_UNDERFUNDED` | Sender doesn't have enough | Check balance before sending |

## Official Documentation
- Assets overview: https://developers.stellar.org/docs/learn/fundamentals/stellar-data-structures/assets
- Issue an asset: https://developers.stellar.org/docs/tokens/how-to-issue-an-asset
- SAC: https://developers.stellar.org/docs/tokens/stellar-asset-contract
