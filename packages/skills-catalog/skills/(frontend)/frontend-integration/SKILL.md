---
name: frontend-integration
description: Frontend integration with Stellar — wallet connections, transaction signing, React patterns, and browser SDK usage. Use when building web apps on Stellar. Do NOT use for backend services or CLI tools.
metadata:
  author: nearx
  version: '1.0.0'
  source: 'Stellar Docs + stellarskills + stellar-dev-skill'
---

# Frontend Integration

> Build web applications on Stellar using JavaScript/TypeScript with wallet connections and transaction signing.

## Stellar Wallets Kit

The Stellar Wallets Kit provides a unified interface for connecting to multiple Stellar wallets:

```bash
npm install @creit.tech/stellar-wallets-kit
```

```typescript
import {
  StellarWalletsKit,
  WalletNetwork,
  allowAllModules,
  FREIGHTER_ID,
} from "@creit.tech/stellar-wallets-kit";

const kit = new StellarWalletsKit({
  network: WalletNetwork.TESTNET,
  selectedWalletId: FREIGHTER_ID,
  modules: allowAllModules(),
});

// Open wallet selection modal
await kit.openModal({
  onWalletSelected: async (option) => {
    kit.setWallet(option.id);
    const { address } = await kit.getAddress();
    console.log("Connected:", address);
  },
});
```

Website: https://stellarwalletskit.dev/

## Freighter Wallet

Freighter is the most popular Stellar browser wallet:

```bash
npm install @stellar/freighter-api
```

```typescript
import freighterApi from "@stellar/freighter-api";

// Check if installed
const isInstalled = await freighterApi.isConnected();

// Request access
const publicKey = await freighterApi.requestAccess();

// Sign transaction
const signedXdr = await freighterApi.signTransaction(
  transaction.toXDR(),
  {
    networkPassphrase: Networks.TESTNET,
  }
);
```

## React Pattern: Connect Wallet + Send Payment

```tsx
import { useState } from "react";
import { Keypair, Networks, TransactionBuilder, Operation, Asset, SorobanRpc } from "@stellar/stellar-sdk";
import freighterApi from "@stellar/freighter-api";

function PaymentApp() {
  const [address, setAddress] = useState("");
  const [status, setStatus] = useState("");

  const connectWallet = async () => {
    const pubKey = await freighterApi.requestAccess();
    setAddress(pubKey);
  };

  const sendPayment = async (destination: string, amount: string) => {
    setStatus("Building transaction...");
    
    const server = new SorobanRpc.Server("https://soroban-testnet.stellar.org");
    const account = await server.getAccount(address);
    
    const tx = new TransactionBuilder(account, {
      fee: "100",
      networkPassphrase: Networks.TESTNET,
    })
      .addOperation(
        Operation.payment({
          destination,
          asset: Asset.native(),
          amount,
        })
      )
      .setTimeout(30)
      .build();

    setStatus("Please sign in wallet...");
    const signedXdr = await freighterApi.signTransaction(tx.toXDR(), {
      networkPassphrase: Networks.TESTNET,
    });

    setStatus("Submitting...");
    const signedTx = TransactionBuilder.fromXDR(signedXdr, Networks.TESTNET);
    const result = await server.sendTransaction(signedTx);
    
    setStatus(`Sent! Hash: ${result.hash}`);
  };

  return (
    <div>
      {!address ? (
        <button onClick={connectWallet}>Connect Wallet</button>
      ) : (
        <div>
          <p>Connected: {address}</p>
          <button onClick={() => sendPayment("GDEST...", "10")}>
            Send 10 XLM
          </button>
          <p>{status}</p>
        </div>
      )}
    </div>
  );
}
```

## Scaffold Stellar

Scaffold Stellar provides starter templates for building on Stellar:

```bash
npx create-soroban-dapp
```

Website: https://scaffoldstellar.org

## SEP-10 Authentication

Web authentication flow for Stellar:

```javascript
import { Utils } from "@stellar/stellar-sdk";

// 1. Fetch challenge from anchor server
const challenge = await fetch(`${anchorUrl}/auth?account=${publicKey}`);
const challengeXdr = await challenge.json();

// 2. Build transaction from challenge
const tx = TransactionBuilder.fromXDR(
  challengeXdr.transaction,
  Networks.TESTNET
);

// 3. Sign with wallet
const signedXdr = await freighterApi.signTransaction(tx.toXDR(), {
  networkPassphrase: Networks.TESTNET,
});

// 4. Submit signed challenge back
const tokenResponse = await fetch(`${anchorUrl}/auth`, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ transaction: signedXdr }),
});

const { token } = await tokenResponse.json(); // JWT token
```

## Official Documentation
- Stellar Wallets Kit: https://stellarwalletskit.dev/
- Freighter: https://www.freighter.app/
- Scaffold Stellar: https://scaffoldstellar.org
- JS SDK: https://developers.stellar.org/docs/tools/sdks/library#javascript-sdk
