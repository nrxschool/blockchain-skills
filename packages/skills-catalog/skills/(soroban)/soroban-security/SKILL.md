---
name: soroban-security
description: Soroban smart contract security patterns — access control, reentrancy, integer safety, auth best practices. Use when auditing or securing Soroban contracts. Do NOT use for general web security or classic Stellar security.
metadata:
  author: nearx
  version: '1.0.0'
  source: 'Stellar Docs + stellarskills + OpenZeppelin'
---

# Soroban Security

> Security patterns and audit checklist for Soroban smart contracts. Follow these before any deployment.

## Access Control

```rust
fn only_admin(env: &Env) {
    let admin: Address = env.storage().instance().get(&DataKey::Admin).unwrap();
    admin.require_auth();
}

// Use in functions
pub fn admin_action(env: Env, admin: Address) {
    Self::only_admin(&env);
    // ... protected logic
}
```

**Rules:**
- Always use `require_auth()` for state-changing operations
- Prefer `require_auth_for_args()` for precise authorization
- Never rely on transaction source as auth (no `msg.sender` equivalent)
- Store admin in Instance storage, not hardcoded

## Reentrancy Protection

Soroban is NOT inherently reentrancy-safe. Use a lock flag:

```rust
#[contracttype]
pub enum DataKey {
    Lock,
    // ...
}

fn acquire_lock(env: &Env) {
    if env.storage().temporary().has(&DataKey::Lock) {
        panic!("reentrant call");
    }
    env.storage().temporary().set(&DataKey::Lock, &true);
}

fn release_lock(env: &Env) {
    env.storage().temporary().remove(&DataKey::Lock);
}

pub fn sensitive_operation(env: Env, user: Address) {
    Self::acquire_lock(&env);
    // ... do work
    Self::release_lock(&env);
}
```

## Integer Safety

Rust panics on overflow in debug, wraps in release. Always use checked arithmetic:

```rust
let new_balance = balance.checked_add(amount).expect("overflow");
let new_supply = supply.checked_sub(burn_amount).expect("underflow");
let result = a.checked_mul(b).expect("multiplication overflow");
let ratio = a.checked_div(b).expect("division by zero");
```

## Storage TTL Management

```rust
// Always extend TTL for critical data
env.storage().persistent().extend_ttl(&key, 100_000, 200_000);

// Instance TTL affects the entire contract
env.storage().instance().extend_ttl(100_000, 200_000);

// Check if data is still live before reading
if env.storage().persistent().has(&key) {
    let val: Type = env.storage().persistent().get(&key).unwrap();
}
```

## Initialization Guard

Prevent re-initialization attacks:

```rust
pub fn initialize(env: Env, admin: Address) {
    if env.storage().instance().has(&DataKey::Admin) {
        panic!("already initialized");
    }
    admin.require_auth();
    env.storage().instance().set(&DataKey::Admin, &admin);
}
```

## Input Validation

```rust
pub fn transfer(env: Env, from: Address, to: Address, amount: i128) {
    from.require_auth();
    
    // Validate inputs
    if amount <= 0 { panic!("amount must be positive"); }
    if from == to { panic!("self-transfer not allowed"); }
    
    let balance: i128 = env.storage().persistent()
        .get(&DataKey::Balance(from.clone()))
        .unwrap_or(0);
    if balance < amount { panic!("insufficient balance"); }
    
    // ... execute transfer
}
```

## Audit Checklist

- [ ] All state-changing functions use `require_auth()`
- [ ] Integer arithmetic uses `checked_*` methods
- [ ] Contract cannot be re-initialized
- [ ] Reentrancy locks on sensitive operations
- [ ] Storage TTL is managed (no silent expiration of critical data)
- [ ] Input validation on all public functions
- [ ] No hardcoded secrets or admin keys
- [ ] Events emitted for all significant state changes
- [ ] Cross-contract calls reviewed for trust assumptions
- [ ] Resource limits tested under load

## Official Documentation
- Soroban authorization: https://developers.stellar.org/docs/build/guides/conventions/authorization
- OpenZeppelin on Stellar: https://www.openzeppelin.com/networks/stellar
- Security best practices: https://developers.stellar.org/docs/build/smart-contracts/overview
