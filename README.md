# 🚀 Nearx Skills — Stellar Network AI Agent Skills

> Knowledge for AI Agents to build on the **Stellar Network**. Curated skills for the Nearx developer community.

[![Stellar](https://img.shields.io/badge/Stellar-Network-blue)](https://stellar.org)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

---

## 🤖 The Problem

AI agents (Claude, Cursor, Copilot, etc.) often hallucinate when asked to build on Stellar. They mix up classic Stellar protocol with EVM concepts, invent non-existent RPC methods, or struggle with Soroban smart contracts.

## 💡 The Solution

**Nearx Skills** provides curated, high-quality Markdown knowledge files that teach AI agents how to build correctly on Stellar. Each skill is dense, factual, and code-heavy — no fluff.

---

## 📚 Skill Index

### ⭐ Stellar Core
| Skill | Description |
|-------|------------|
| [accounts](packages/skills-catalog/skills/(stellar-core)/accounts/SKILL.md) | Keypairs, funding, signers, sponsorship, minimum balances |
| [assets](packages/skills-catalog/skills/(stellar-core)/assets/SKILL.md) | Asset issuance, trustlines, USDC, Stellar Asset Contract (SAC) |
| [operations](packages/skills-catalog/skills/(stellar-core)/operations/SKILL.md) | All transaction operations — Payment, ManageOffer, PathPayment |
| [fees](packages/skills-catalog/skills/(stellar-core)/fees/SKILL.md) | Base fees, resource fees, surge pricing |

### 🔧 Soroban Smart Contracts
| Skill | Description |
|-------|------------|
| [soroban-contracts](packages/skills-catalog/skills/(soroban)/soroban-contracts/SKILL.md) | Rust/WASM contracts, storage, auth, deployment |
| [soroban-security](packages/skills-catalog/skills/(soroban)/soroban-security/SKILL.md) | Access control, reentrancy, integer safety |
| [soroban-testing](packages/skills-catalog/skills/(soroban)/soroban-testing/SKILL.md) | Unit tests, testnet, local development |

### 🌐 APIs & Data
| Skill | Description |
|-------|------------|
| [stellar-rpc](packages/skills-catalog/skills/(apis)/stellar-rpc/SKILL.md) | JSON-RPC API — simulate, send, getEvents |
| [horizon-legacy](packages/skills-catalog/skills/(apis)/horizon-legacy/SKILL.md) | Horizon REST API (deprecated, for legacy) |

### 🖥️ Frontend & Wallets
| Skill | Description |
|-------|------------|
| [frontend-integration](packages/skills-catalog/skills/(frontend)/frontend-integration/SKILL.md) | Wallet connections, React patterns, Freighter |

### 💰 Payments & Anchors
| Skill | Description |
|-------|------------|
| [x402-stellar](packages/skills-catalog/skills/(payments)/x402-stellar/SKILL.md) | HTTP 402 micropayments, machine payments |
| [seps-anchors](packages/skills-catalog/skills/(payments)/seps-anchors/SKILL.md) | SEP standards, fiat on/off-ramps |

### 🛠️ Developer Tools
| Skill | Description |
|-------|------------|
| [stellar-cli](packages/skills-catalog/skills/(tools)/stellar-cli/SKILL.md) | CLI for contract dev, key management |
| [stellar-sdks](packages/skills-catalog/skills/(tools)/stellar-sdks/SKILL.md) | JavaScript, Python, Rust SDKs |

### 📊 DeFi
| Skill | Description |
|-------|------------|
| [dex-amm](packages/skills-catalog/skills/(defi)/dex-amm/SKILL.md) | Built-in DEX, AMM, liquidity pools |

### 🤖 AI & Agent Integration
| Skill | Description |
|-------|------------|
| [mcp-servers](packages/skills-catalog/skills/(ai-agents)/mcp-servers/SKILL.md) | MCP servers, AI-assisted Stellar dev |

### 🔒 Security
| Skill | Description |
|-------|------------|
| [smart-contract-security](packages/skills-catalog/skills/(security)/smart-contract-security/SKILL.md) | OpenZeppelin, audit checklists |

### 🌍 Community
| Skill | Description |
|-------|------------|
| [nearx-contributing-guide](packages/skills-catalog/skills/(community)/nearx-contributing-guide/SKILL.md) | How to contribute new skills |

---

## ⚡ Quick Start

### Using Skills with AI Agents

Give your AI agent a skill URL in your prompt:

```
Read https://raw.githubusercontent.com/nearxdev/skills/main/packages/skills-catalog/skills/(soroban)/soroban-contracts/SKILL.md and help me write a token contract.
```

### Installing as a Plugin (Claude Code)

```bash
claude plugin install https://github.com/nearxdev/skills
```

### Using the CLI

```bash
# Install skills for your AI agent
npx @nearx/skills install

# List available skills
npx @nearx/skills list
```

---

## 🤝 Contributing

We want this to be the knowledge hub for Stellar developers in the Nearx community. **Contributions are welcome!**

1. Read the [contributing guide](packages/skills-catalog/skills/(community)/nearx-contributing-guide/SKILL.md)
2. Fork this repository
3. Create a new skill in the appropriate category
4. Submit a Pull Request

### Suggested Skills to Create
- `scaffold-stellar` — Scaffold Stellar starter templates
- `passkey-wallets` — Smart accounts with passkeys
- `soroban-advanced-patterns` — Factory, proxy, upgradeability
- `hackathon-guide` — Stellar hackathon tips and FAQ

---

## 📌 Stellar Resources

| Resource | Link |
|----------|------|
| **Stellar Docs** | https://developers.stellar.org/ |
| **Stellar Lab** | https://lab.stellar.org |
| **Stellar Expert** | https://stellar.expert |
| **Stellar Discord** | https://discord.gg/stellar |
| **SDKs** | https://developers.stellar.org/docs/tools/sdks |
| **LLMs.txt** | https://developers.stellar.org/llms.txt |
| **Stellarskills** | https://github.com/ggoldani/stellarskills |
| **Stellar Dev Skill** | https://github.com/stellar/stellar-dev-skill |

---

## 📄 License

MIT © [Nearx](https://github.com/nearxdev)

Built with ❤️ by the Nearx community for Stellar developers.
