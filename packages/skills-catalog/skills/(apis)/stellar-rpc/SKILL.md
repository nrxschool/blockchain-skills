---
name: stellar-rpc
description: Stellar RPC JSON-RPC API — simulateTransaction, sendTransaction, getTransaction, getEvents, getLedgerEntries. Use when interacting with the Stellar network programmatically. Do NOT use for Horizon REST API.
metadata:
  author: nearx
  version: '1.0.0'
  source: 'Stellar Docs + stellarskills'
---

# Stellar RPC

> Stellar RPC (JSON-RPC) is the **supported** API for smart contracts and the direction of travel for all on-chain data access. Prefer Stellar RPC over Horizon for new projects.

## Endpoints

Pick an RPC endpoint from: https://developers.stellar.org/docs/data/apis/rpc/providers

| Network | Example Endpoint |
|---------|-----------------|
| **Testnet** | `https://soroban-testnet.stellar.org` (SDF, dev only) |
| **Mainnet** | Use a provider (Blockdaemon, Validation Cloud, QuickNode, etc.) |

## JavaScript SDK Setup

```javascript
import { SorobanRpc } from "@stellar/stellar-sdk";

const server = new SorobanRpc.Server("https://soroban-testnet.stellar.org");
```

## Core Methods

### simulateTransaction
Dry-run a transaction to get resource estimates and results:

```javascript
const sim = await server.simulateTransaction(tx);
if (SorobanRpc.Api.isSimulationError(sim)) {
  console.error("Simulation failed:", sim.error);
} else {
  console.log("Min resource fee:", sim.minResourceFee);
  console.log("Result:", sim.result);
}
```

### sendTransaction
Submit a signed transaction:

```javascript
const preparedTx = SorobanRpc.assembleTransaction(tx, sim);
preparedTx.sign(keypair);
const sendResult = await server.sendTransaction(preparedTx);

// Poll for completion
if (sendResult.status === "PENDING") {
  let getResult;
  do {
    await new Promise((r) => setTimeout(r, 1000));
    getResult = await server.getTransaction(sendResult.hash);
  } while (getResult.status === "NOT_FOUND");
  
  if (getResult.status === "SUCCESS") {
    console.log("Transaction succeeded!");
  } else {
    console.error("Transaction failed:", getResult);
  }
}
```

### getTransaction
Get transaction result by hash:

```javascript
const result = await server.getTransaction(txHash);
// status: "SUCCESS" | "FAILED" | "NOT_FOUND"
```

### getEvents
Query contract events:

```javascript
const events = await server.getEvents({
  startLedger: ledgerNumber,
  filters: [
    {
      type: "contract",
      contractIds: [contractId],
      topics: [["AAAADwAAAAh0cmFuc2Zlcg=="]], // base64-encoded topic
    },
  ],
});
```

### getLedgerEntries
Read raw ledger data:

```javascript
const entries = await server.getLedgerEntries(keys);
```

### getLatestLedger
Get current network state:

```javascript
const latest = await server.getLatestLedger();
console.log("Ledger:", latest.sequence);
```

## Full Workflow: Simulate → Assemble → Sign → Send → Poll

```javascript
import {
  Contract, SorobanRpc, TransactionBuilder, Networks, nativeToScVal, Keypair
} from "@stellar/stellar-sdk";

const server = new SorobanRpc.Server("https://soroban-testnet.stellar.org");
const keypair = Keypair.fromSecret(secretKey);
const account = await server.getAccount(keypair.publicKey());
const contract = new Contract(contractId);

// 1. Build transaction
const tx = new TransactionBuilder(account, {
  fee: "1000000",
  networkPassphrase: Networks.TESTNET,
})
  .addOperation(contract.call("my_function", nativeToScVal(arg1, { type: "address" })))
  .setTimeout(30)
  .build();

// 2. Simulate
const sim = await server.simulateTransaction(tx);
if (SorobanRpc.Api.isSimulationError(sim)) throw new Error(sim.error);

// 3. Assemble (adds resource info from simulation)
const preparedTx = SorobanRpc.assembleTransaction(tx, sim);

// 4. Sign
preparedTx.sign(keypair);

// 5. Send
const sendResult = await server.sendTransaction(preparedTx);

// 6. Poll for result
let result;
do {
  await new Promise((r) => setTimeout(r, 1000));
  result = await server.getTransaction(sendResult.hash);
} while (result.status === "NOT_FOUND");

console.log("Final status:", result.status);
```

## Common Errors

| Error | Cause | Fix |
|-------|-------|-----|
| Simulation error `HostError` | Contract logic failure | Check contract code and inputs |
| `txn_submit_failed` | Transaction rejected | Check fee, sequence number, auth |
| Status `NOT_FOUND` forever | Transaction dropped | Resubmit with higher fee |

## Official Documentation
- Stellar RPC overview: https://developers.stellar.org/docs/data/apis/rpc
- RPC methods: https://developers.stellar.org/docs/data/apis/rpc/methods
- RPC providers: https://developers.stellar.org/docs/data/apis/rpc/providers
- Migrate from Horizon: https://developers.stellar.org/docs/data/apis/migrate-from-horizon-to-rpc
