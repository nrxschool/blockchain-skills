---
name: soroban-testing
description: Testing Soroban smart contracts — unit tests, integration tests, testnet deployment, Stellar CLI testing. Use when writing tests or debugging contracts. Do NOT use for frontend testing or general test frameworks.
metadata:
  author: nearx
  version: '1.0.0'
  source: 'Stellar Docs + stellarskills'
---

# Soroban Testing

> Test Soroban contracts using Rust's built-in test framework with soroban-sdk testutils. Always test before deploying.

## Unit Tests

```rust
#![cfg(test)]
use super::*;
use soroban_sdk::{testutils::Address as _, Address, Env};

#[test]
fn test_initialize() {
    let env = Env::default();
    env.mock_all_auths(); // Auto-approve all auth calls in tests
    
    let contract_id = env.register(MyContract, ());
    let client = MyContractClient::new(&env, &contract_id);
    
    let admin = Address::generate(&env);
    client.initialize(&admin);
    
    // Verify state
    assert_eq!(client.get_balance(&admin), 0);
}

#[test]
fn test_set_balance() {
    let env = Env::default();
    env.mock_all_auths();
    
    let contract_id = env.register(MyContract, ());
    let client = MyContractClient::new(&env, &contract_id);
    
    let admin = Address::generate(&env);
    let user = Address::generate(&env);
    
    client.initialize(&admin);
    client.set_balance(&admin, &user, &1000);
    
    assert_eq!(client.get_balance(&user), 1000);
}

#[test]
#[should_panic(expected = "not admin")]
fn test_unauthorized_set_balance() {
    let env = Env::default();
    env.mock_all_auths();
    
    let contract_id = env.register(MyContract, ());
    let client = MyContractClient::new(&env, &contract_id);
    
    let admin = Address::generate(&env);
    let attacker = Address::generate(&env);
    let user = Address::generate(&env);
    
    client.initialize(&admin);
    client.set_balance(&attacker, &user, &1000); // Should panic
}
```

## Testing with Tokens (SAC)

```rust
#[test]
fn test_token_transfer() {
    let env = Env::default();
    env.mock_all_auths();
    
    // Register a test token
    let token_admin = Address::generate(&env);
    let (token_id, token_admin_client) = create_token(&env, &token_admin);
    let token_client = token::Client::new(&env, &token_id);
    
    // Mint tokens to user
    let user = Address::generate(&env);
    token_admin_client.mint(&user, &10000);
    
    assert_eq!(token_client.balance(&user), 10000);
}
```

## Running Tests

```bash
# Run all tests
cargo test

# Run specific test
cargo test test_initialize

# Run with output
cargo test -- --nocapture

# Run in release mode (matches on-chain behavior)
cargo test --release
```

## Testing on Testnet

```bash
# Configure testnet identity
stellar keys generate my-test-key --network testnet --fund

# Deploy contract
stellar contract deploy \
  --wasm target/wasm32-unknown-unknown/release/my_contract.wasm \
  --source my-test-key \
  --network testnet

# Invoke and test
stellar contract invoke \
  --id CONTRACT_ID \
  --source my-test-key \
  --network testnet \
  -- \
  initialize \
  --admin $(stellar keys address my-test-key)
```

## Local Development with Quickstart

```bash
# Run local Stellar node with Docker
docker run --rm -it \
  -p 8000:8000 \
  --name stellar \
  stellar/quickstart:latest \
  --testnet \
  --enable-soroban-rpc
```

## Auth Verification in Tests

```rust
use soroban_sdk::testutils::AuthorizedFunction;

#[test]
fn test_auth_required() {
    let env = Env::default();
    // Don't mock auths — test real auth flow
    
    let contract_id = env.register(MyContract, ());
    let client = MyContractClient::new(&env, &contract_id);
    
    let admin = Address::generate(&env);
    client.initialize(&admin);
    
    // Verify which auths were required
    let auths = env.auths();
    assert_eq!(auths.len(), 1);
}
```

## Tips
1. **Always test in release mode** before deploying — debug and release can behave differently (integer overflow)
2. **Testnet resets periodically** — don't store testnet state long-term
3. **Use `env.mock_all_auths()`** for unit tests, but also test real auth flows
4. **Test edge cases**: zero amounts, max values, expired storage, reentrancy

## Official Documentation
- Getting started: https://developers.stellar.org/docs/build/smart-contracts/getting-started
- Testing: https://developers.stellar.org/docs/build/smart-contracts/getting-started/storing-data
- Stellar CLI: https://developers.stellar.org/docs/tools/cli
- Quickstart (Docker): https://developers.stellar.org/docs/tools/quickstart
