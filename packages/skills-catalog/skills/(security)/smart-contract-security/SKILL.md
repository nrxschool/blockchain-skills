---
name: smart-contract-security
description: Smart contract security on Stellar — OpenZeppelin contracts, audit patterns, common vulnerabilities. Use when securing Soroban contracts or preparing for audits. Do NOT use for general web security.
metadata:
  author: nearx
  version: '1.0.0'
  source: 'OpenZeppelin + Stellar Docs'
---

# Smart Contract Security

> Security best practices for Soroban smart contracts, including OpenZeppelin contracts on Stellar.

## OpenZeppelin on Stellar

OpenZeppelin provides audited, reusable smart contract building blocks for Soroban:

- Website: https://www.openzeppelin.com/networks/stellar
- Skills: https://github.com/OpenZeppelin/openzeppelin-skills

### Available Contracts

| Contract | Purpose |
|----------|---------|
| **Access Control** | Role-based access management |
| **Pausable** | Emergency pause mechanism |
| **Upgradeable** | Safe upgrade patterns |
| **Token Standards** | SEP-41 compliant tokens |

## Security Checklist for Soroban

### Before Deployment
- [ ] All state-changing functions require `require_auth()`
- [ ] Integer arithmetic uses `checked_*` methods
- [ ] Contract cannot be re-initialized
- [ ] Reentrancy protection on sensitive operations
- [ ] Storage TTL managed — no silent expiration of critical data
- [ ] Input validation on all public functions
- [ ] No hardcoded secrets or admin keys
- [ ] Events emitted for all significant state changes
- [ ] Cross-contract calls reviewed for trust assumptions
- [ ] Tested under resource limits

### Common Vulnerabilities

| Vulnerability | Impact | Prevention |
|--------------|--------|-----------|
| **Missing auth** | Unauthorized state changes | Always `require_auth()` |
| **Integer overflow** | Incorrect balances | Use `checked_add/sub/mul/div` |
| **Re-initialization** | Admin takeover | Guard `initialize()` with existence check |
| **Reentrancy** | Fund drainage | Use lock flags |
| **Expired storage** | Data loss | Extend TTL proactively |
| **Unvalidated inputs** | Logic bypass | Validate all parameters |
| **Unchecked return values** | Silent failures | Handle all `Option`/`Result` |

### Access Control Pattern

```rust
use soroban_sdk::{Address, Env};

#[contracttype]
pub enum Role {
    Admin,
    Minter,
    Pauser,
}

fn check_role(env: &Env, who: &Address, role: Role) {
    who.require_auth();
    let has_role: bool = env.storage()
        .instance()
        .get(&(role, who.clone()))
        .unwrap_or(false);
    if !has_role { panic!("unauthorized"); }
}
```

### Pausable Pattern

```rust
fn require_not_paused(env: &Env) {
    let paused: bool = env.storage()
        .instance()
        .get(&DataKey::Paused)
        .unwrap_or(false);
    if paused { panic!("contract is paused"); }
}

pub fn pause(env: Env, admin: Address) {
    Self::check_role(&env, &admin, Role::Pauser);
    env.storage().instance().set(&DataKey::Paused, &true);
}
```

### Safe Math

```rust
// ALWAYS use checked arithmetic in production
let new_balance = balance
    .checked_add(amount)
    .expect("balance overflow");

let fee = total
    .checked_mul(fee_bps as i128)
    .and_then(|v| v.checked_div(10_000))
    .expect("fee calculation overflow");
```

## Contract Accounts

Stellar supports contract-based accounts for advanced patterns:
- Docs: https://developers.stellar.org/docs/build/guides/conventions/contract-accounts
- Authorization: https://developers.stellar.org/docs/build/guides/conventions/authorization

## Audit Resources
- OpenZeppelin on Stellar: https://www.openzeppelin.com/networks/stellar
- Stellar security docs: https://developers.stellar.org/docs/build/smart-contracts/overview
