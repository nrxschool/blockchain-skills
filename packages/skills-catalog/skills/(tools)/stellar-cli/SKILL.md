---
name: stellar-cli
description: Stellar CLI — contract management, key management, network configuration, and development workflows. Use when working with the Stellar CLI tool. Do NOT use for JavaScript SDK or frontend development.
metadata:
  author: nearx
  version: '1.0.0'
  source: 'Stellar Docs + stellarskills'
---

# Stellar CLI

> The Stellar CLI is the primary tool for Soroban contract development — building, deploying, testing, and invoking contracts.

## Installation

```bash
# Install via cargo
cargo install --locked stellar-cli --features opt

# Verify
stellar --version
```

## Key Management

```bash
# Generate a new key (funds on testnet automatically)
stellar keys generate my-key --network testnet --fund

# List keys
stellar keys ls

# Get public address
stellar keys address my-key

# Get secret key (careful!)
stellar keys show my-key
```

## Network Configuration

```bash
# Networks are preconfigured: testnet, mainnet, futurenet
# Or add custom network
stellar network add my-network \
  --rpc-url https://soroban-testnet.stellar.org \
  --network-passphrase "Test SDF Network ; September 2015"
```

## Contract Workflow

```bash
# 1. Create new project
stellar contract init my_project
cd my_project

# 2. Build
stellar contract build

# 3. Optimize (smaller WASM, lower deploy fees)
stellar contract optimize \
  --wasm target/wasm32-unknown-unknown/release/my_project.wasm

# 4. Deploy
stellar contract deploy \
  --wasm target/wasm32-unknown-unknown/release/my_project.wasm \
  --source my-key \
  --network testnet
# Returns: CONTRACT_ID (C...)

# 5. Invoke
stellar contract invoke \
  --id CONTRACT_ID \
  --source my-key \
  --network testnet \
  -- \
  function_name \
  --arg1 value1

# 6. Read contract data
stellar contract read \
  --id CONTRACT_ID \
  --network testnet
```

## Contract Bindings

Generate TypeScript/JavaScript client for your contract:

```bash
stellar contract bindings typescript \
  --contract-id CONTRACT_ID \
  --network testnet \
  --output-dir ./client
```

## Useful Commands

```bash
# Check account balance
stellar account balance \
  --address GADDRESS... \
  --network testnet

# Fund testnet account
stellar account fund --address GADDRESS... --network testnet

# Check network settings (resource limits, fees)
stellar network settings --network testnet

# Extend contract TTL
stellar contract extend \
  --id CONTRACT_ID \
  --durability persistent \
  --ledgers-to-extend 100000 \
  --source my-key \
  --network testnet

# Restore expired contract data
stellar contract restore \
  --id CONTRACT_ID \
  --source my-key \
  --network testnet
```

## Tips
1. Always `stellar contract optimize` before deploying to save on fees
2. Use `--fund` when generating testnet keys for automatic Friendbot funding
3. The `--` separator is required before contract function arguments
4. Use `stellar contract read` to inspect on-chain contract state

## Official Documentation
- Stellar CLI: https://developers.stellar.org/docs/tools/cli
- CLI reference: https://developers.stellar.org/docs/tools/cli/stellar-cli
