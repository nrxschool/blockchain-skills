---
name: stellar-accounts
description: Stellar account model — keypairs, funding, signers, sponsorship, and minimum balances. Use when creating accounts, managing signers, or understanding the Stellar account model. Do NOT use for Soroban contract accounts or smart wallet patterns.
metadata:
  author: nearx
  version: '1.0.0'
  source: 'Stellar Docs + stellarskills'
---

# Stellar Accounts

> Stellar accounts are the foundation of the network. Every participant must have a funded account before they can transact.

## Key Concepts

### Account Creation
Accounts must be **explicitly created and funded** on Stellar. You cannot send XLM to a keypair that has never been funded.

```javascript
import { Keypair, Networks, TransactionBuilder, Operation, Horizon } from "@stellar/stellar-sdk";

// Generate a new keypair
const pair = Keypair.random();
console.log("Public Key:", pair.publicKey());   // G...
console.log("Secret Key:", pair.secret());       // S...

// Fund on testnet via Friendbot
const response = await fetch(
  `https://friendbot.stellar.org?addr=${pair.publicKey()}`
);

// Create account on mainnet (requires existing funded account)
const server = new Horizon.Server("https://horizon.stellar.org");
const sourceAccount = await server.loadAccount(sourcePublicKey);

const tx = new TransactionBuilder(sourceAccount, {
  fee: "100",
  networkPassphrase: Networks.PUBLIC,
})
  .addOperation(
    Operation.createAccount({
      destination: pair.publicKey(),
      startingBalance: "2", // minimum ~1 XLM + reserves
    })
  )
  .setTimeout(30)
  .build();

tx.sign(sourceKeypair);
await server.submitTransaction(tx);
```

### Minimum Balance
Every Stellar account requires a **minimum balance** (base reserve). Each additional entry (trustline, offer, signer, data entry) increases the minimum.

**Current formula**: `(2 + numEntries) × baseReserve`

Check current values at: https://developers.stellar.org/docs/networks/resource-limits-fees

### Signers & Multi-sig
Accounts can have multiple signers with different weights and thresholds:

```javascript
const tx = new TransactionBuilder(account, {
  fee: "100",
  networkPassphrase: Networks.PUBLIC,
})
  .addOperation(
    Operation.setOptions({
      signer: {
        ed25519PublicKey: additionalSignerPublicKey,
        weight: 1,
      },
      medThreshold: 2,  // Requires 2 signers for payments
      highThreshold: 3, // Requires 3 signers for account changes
    })
  )
  .setTimeout(30)
  .build();
```

### Account Sponsorship
Sponsors can pay the minimum balance reserves for other accounts:

```javascript
const tx = new TransactionBuilder(sponsorAccount, {
  fee: "100",
  networkPassphrase: Networks.PUBLIC,
})
  .addOperation(Operation.beginSponsoringFutureReserves({ sponsoredId: newAccount.publicKey() }))
  .addOperation(Operation.createAccount({ destination: newAccount.publicKey(), startingBalance: "0" }))
  .addOperation(Operation.endSponsoringFutureReserves({ source: newAccount.publicKey() }))
  .setTimeout(30)
  .build();

tx.sign(sponsorKeypair, newAccountKeypair);
```

### Account Merge
Remove an account and transfer remaining XLM to a destination:

```javascript
.addOperation(Operation.accountMerge({ destination: destinationPublicKey }))
```

## Network Passphrases
Every transaction must be signed with the correct network passphrase:

| Network | Passphrase |
|---------|-----------|
| **Mainnet** | `Public Global Stellar Network ; September 2015` |
| **Testnet** | `Test SDF Network ; September 2015` |

## Common Errors

| Error | Cause | Fix |
|-------|-------|-----|
| `CREATE_ACCOUNT_UNDERFUNDED` | Source doesn't have enough XLM | Ensure source has enough balance + reserves |
| `CREATE_ACCOUNT_ALREADY_EXIST` | Destination account already exists | Check if account exists before creating |
| `ACCOUNT_MERGE_HAS_SUB_ENTRIES` | Account has trustlines/offers/data | Remove all entries before merging |
| `SET_OPTIONS_BAD_SIGNER` | Invalid signer key | Verify the signer public key format |
| `tx_bad_seq` | Wrong sequence number | Reload account from network before building tx |

## Official Documentation
- Accounts: https://developers.stellar.org/docs/learn/fundamentals/stellar-data-structures/accounts
- Operations: https://developers.stellar.org/docs/learn/fundamentals/transactions/list-of-operations
- Sponsorship: https://developers.stellar.org/docs/learn/encyclopedia/transactions-specialized/sponsored-reserves
