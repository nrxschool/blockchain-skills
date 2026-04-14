---
name: soroban-contracts
description: Soroban smart contract development — Rust/WASM contracts, storage, auth, cross-contract calls, deployment. Use when writing, deploying, or invoking Soroban contracts. Do NOT use for classic Stellar operations or frontend integration.
metadata:
  author: nearx
  version: '1.0.0'
  source: 'Stellar Docs + stellarskills'
---

# Soroban Smart Contracts

> Soroban is Stellar's smart contract platform. Contracts are written in Rust, compiled to WASM, and executed in a deterministic sandbox on-chain.

## Key Differences from EVM
- **No Solidity** — Rust only
- **No global mutable state** — contracts use typed storage (Instance, Persistent, Temporary)
- **Explicit auth model** — `env.require_auth(&address)` rather than `msg.sender`
- **Resource budget** — every invocation has CPU/memory/ledger read-write limits
- **Classic assets accessible via SAC** (Stellar Asset Contract)

## Project Setup

```bash
# Install Stellar CLI
cargo install --locked stellar-cli --features opt

# Create new contract project
stellar contract init my_contract
cd my_contract
```

```toml
# Cargo.toml
[lib]
crate-type = ["cdylib"]

[dependencies]
soroban-sdk = { version = "25.3.0", features = ["alloc"] }

[dev-dependencies]
soroban-sdk = { version = "25.3.0", features = ["testutils", "alloc"] }
```

> Pin soroban-sdk to the release matching your network. Verify at: https://docs.rs/soroban-sdk/latest/soroban_sdk/

## Contract Structure

```rust
#![no_std]
use soroban_sdk::{contract, contractimpl, contracttype, Address, Env, Symbol, symbol_short};

#[contracttype]
pub enum DataKey {
    Balance(Address),
    Admin,
    TotalSupply,
}

#[contract]
pub struct MyContract;

#[contractimpl]
impl MyContract {
    pub fn initialize(env: Env, admin: Address) {
        admin.require_auth();
        env.storage().instance().set(&DataKey::Admin, &admin);
        env.storage().instance().set(&DataKey::TotalSupply, &0_i128);
    }

    pub fn get_balance(env: Env, address: Address) -> i128 {
        env.storage()
            .persistent()
            .get(&DataKey::Balance(address))
            .unwrap_or(0)
    }

    pub fn set_balance(env: Env, admin: Address, target: Address, amount: i128) {
        admin.require_auth();
        let stored_admin: Address = env.storage().instance().get(&DataKey::Admin).unwrap();
        if admin != stored_admin { panic!("not admin"); }
        env.storage().persistent().set(&DataKey::Balance(target), &amount);
    }
}
```

## Storage Types

| Type | TTL | Use Case |
|------|-----|----------|
| **Instance** | Tied to contract | Config, admin, small global state |
| **Persistent** | Survives, needs rent | User balances, important data |
| **Temporary** | Auto-expires | Locks, nonces, short-lived data |

```rust
// Instance storage — lives as long as the contract
env.storage().instance().set(&key, &value);
env.storage().instance().get::<_, Type>(&key);

// Persistent storage — survives but needs TTL extension
env.storage().persistent().set(&key, &value);
env.storage().persistent().extend_ttl(&key, 100, 200); // min_ttl, max_ttl

// Temporary storage — auto-cleaned
env.storage().temporary().set(&key, &value);
```

## Authentication

```rust
// Basic auth — require caller has authorized this invocation
caller.require_auth();

// Auth with specific args (more secure)
caller.require_auth_for_args((amount, destination).into_val(&env));
```

## Events

```rust
env.events().publish(
    (symbol_short!("transfer"),),       // topics (up to 4)
    (from.clone(), to.clone(), amount), // data
);
```

## Cross-Contract Calls

```rust
use soroban_sdk::token;

let token_client = token::Client::new(&env, &token_contract_id);
let balance = token_client.balance(&user);
token_client.transfer(&from, &to, &amount);
```

## Build & Deploy

```bash
# Build
stellar contract build

# Optimize (smaller WASM)
stellar contract optimize --wasm target/wasm32-unknown-unknown/release/my_contract.wasm

# Deploy to testnet
stellar contract deploy \
  --wasm target/wasm32-unknown-unknown/release/my_contract.wasm \
  --source my-key \
  --network testnet

# Invoke
stellar contract invoke \
  --id CONTRACT_ID \
  --source my-key \
  --network testnet \
  -- \
  initialize \
  --admin GADMIN_ADDRESS
```

## Invoke via JavaScript

```javascript
import { Contract, SorobanRpc, TransactionBuilder, Networks, nativeToScVal } from "@stellar/stellar-sdk";

const server = new SorobanRpc.Server("https://soroban-testnet.stellar.org");
const contract = new Contract(contractId);

const tx = new TransactionBuilder(account, {
  fee: "1000000",
  networkPassphrase: Networks.TESTNET,
})
  .addOperation(
    contract.call("get_balance", nativeToScVal(userAddress, { type: "address" }))
  )
  .setTimeout(30)
  .build();

// Always simulate first
const sim = await server.simulateTransaction(tx);
if (SorobanRpc.Api.isSimulationError(sim)) throw new Error(sim.error);

const preparedTx = SorobanRpc.assembleTransaction(tx, sim);
preparedTx.sign(keypair);
const result = await server.sendTransaction(preparedTx);
```

## Common Errors

| Error | Cause | Fix |
|-------|-------|-----|
| `wasm_vm_error` | Contract panicked or exceeded budget | Check logic, reduce resource usage |
| `auth_not_authorized` | Missing `require_auth` signature | Ensure invoker signed and auth matches |
| `storage_not_live` | Accessing expired entry | Extend TTL before reading |
| `invoke_error: value missing` | Storage key not set | Use `.unwrap_or()` or check `.has()` |
| Simulation succeeds, submission fails | State changed between sim and submit | Re-simulate with latest ledger |

## Official Documentation
- Soroban overview: https://developers.stellar.org/docs/build/smart-contracts/overview
- Storing data: https://developers.stellar.org/docs/build/smart-contracts/getting-started/storing-data
- soroban-sdk: https://docs.rs/soroban-sdk/latest/soroban_sdk/
