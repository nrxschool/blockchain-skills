---
name: stellar-sdks
description: Stellar SDKs — JavaScript, Python, Go, Rust client libraries for interacting with the Stellar network. Use when choosing or using an SDK. Do NOT use for contract development (see soroban-contracts).
metadata:
  author: nearx
  version: '1.0.0'
  source: 'Stellar Docs + stellarskills'
---

# Stellar SDKs

> Stellar provides official SDKs for JavaScript, Python, Go, and community SDKs for other languages.

## JavaScript SDK (Primary)

The most complete and actively maintained SDK:

```bash
npm install @stellar/stellar-sdk
```

```javascript
import {
  Keypair,
  Networks,
  TransactionBuilder,
  Operation,
  Asset,
  SorobanRpc,
  Horizon,
  Contract,
  nativeToScVal,
} from "@stellar/stellar-sdk";
```

Check latest release: https://github.com/stellar/js-stellar-sdk/releases

### Key Classes

| Class | Purpose |
|-------|---------|
| `Keypair` | Key generation and signing |
| `TransactionBuilder` | Build transactions |
| `Operation` | Create operations |
| `Asset` | Represent assets |
| `SorobanRpc.Server` | Interact with Stellar RPC |
| `Horizon.Server` | Interact with Horizon (legacy) |
| `Contract` | Interact with Soroban contracts |
| `nativeToScVal` | Convert JS values to Soroban types |

### Common Patterns

```javascript
// Convert between types
import { nativeToScVal, scValToNative, Address } from "@stellar/stellar-sdk";

// JS → Soroban
const scAddress = nativeToScVal(publicKey, { type: "address" });
const scAmount = nativeToScVal(1000n, { type: "i128" });
const scString = nativeToScVal("hello", { type: "string" });
const scBool = nativeToScVal(true, { type: "bool" });

// Soroban → JS
const jsValue = scValToNative(scVal);
```

## Python SDK

```bash
pip install stellar-sdk
```

```python
from stellar_sdk import Keypair, Network, Server, TransactionBuilder

server = Server("https://horizon-testnet.stellar.org")
keypair = Keypair.random()

# Fund on testnet
import requests
requests.get(f"https://friendbot.stellar.org?addr={keypair.public_key}")

# Build transaction
account = server.load_account(keypair.public_key)
tx = (
    TransactionBuilder(
        source_account=account,
        network_passphrase=Network.TESTNET_NETWORK_PASSPHRASE,
        base_fee=100,
    )
    .append_payment_op(destination="GDEST...", asset_code="XLM", amount="10")
    .set_timeout(30)
    .build()
)
tx.sign(keypair)
response = server.submit_transaction(tx)
```

GitHub: https://github.com/StellarCN/py-stellar-base

## Rust SDK

For Soroban contract development, use `soroban-sdk`. For client-side Rust:

```toml
[dependencies]
stellar-sdk = "0.4"
```

## SDK Selection Guide

| Use Case | Recommended SDK |
|----------|----------------|
| Web frontend | JavaScript (`@stellar/stellar-sdk`) |
| Node.js backend | JavaScript (`@stellar/stellar-sdk`) |
| Data analysis / scripts | Python (`stellar-sdk`) |
| Smart contracts | Rust (`soroban-sdk`) |
| System programming | Rust or Go |

## Official Documentation
- SDKs overview: https://developers.stellar.org/docs/tools/sdks
- JavaScript SDK: https://github.com/stellar/js-stellar-sdk
- Python SDK: https://github.com/StellarCN/py-stellar-base
- Stellar Lab: https://developers.stellar.org/docs/tools/lab
