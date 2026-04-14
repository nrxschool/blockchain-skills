---
name: nearx-contributing-guide
description: Guide for Nearx community members to contribute skills to this repository. Use when adding a new skill or understanding the contribution process. Do NOT use for using existing skills.
metadata:
  author: nearx
  version: '1.0.0'
  source: 'Nearx'
---

# Nearx Contributing Guide

> This repository is maintained by the Nearx community. Students and developers are encouraged to contribute new skills and improve existing ones.

## How Skills Work

A skill is a `.md` file that teaches AI agents (Claude, Cursor, Copilot, etc.) about a specific topic. When an agent reads a skill, it gains knowledge about that domain and can help you code better.

## Skill Structure

Each skill lives in its own folder under a category:

```
packages/skills-catalog/skills/
  (stellar-core)/
    accounts/
      SKILL.md          ← The skill file
    assets/
      SKILL.md
  (soroban)/
    soroban-contracts/
      SKILL.md
```

## Creating a New Skill

### 1. Choose a Category

| Category | Topics |
|----------|--------|
| `(stellar-core)` | Protocol fundamentals, accounts, assets |
| `(soroban)` | Smart contracts, storage, testing |
| `(apis)` | RPC, Horizon, data access |
| `(frontend)` | Wallets, browser SDK, React |
| `(payments)` | x402, SEPs, anchors |
| `(tools)` | CLI, SDKs, Stellar Lab |
| `(defi)` | DEX, AMM, DeFi protocols |
| `(ai-agents)` | MCP servers, AI integrations |
| `(security)` | Auditing, OpenZeppelin, patterns |
| `(community)` | Guides, learning, resources |

### 2. Create the SKILL.md

```markdown
---
name: my-skill-name
description: What it does. Use when [triggers]. Do NOT use for [exclusions].
metadata:
  author: your-github-username
  version: '1.0.0'
  source: 'Source reference'
---

# Skill Title

> One-line summary of what this skill teaches.

## Content

Dense, factual content with code examples.
No fluff. No marketing copy.

## Common Errors

| Error | Cause | Fix |
|-------|-------|-----|
| ... | ... | ... |

## Official Documentation
- Link 1: https://...
- Link 2: https://...
```

### 3. Quality Rules

1. **No fluff** — Agents don't need marketing copy
2. **Dense & factual** — Prioritize code snippets, architectural rules, exact endpoints
3. **Current & runnable** — Ensure code compiles against latest SDK versions
4. **Verified by official docs** — Every feature must be documented in official sources
5. **Include Common Errors** — Map error codes to fixes
6. **Under 500 lines** — Keep it focused; offload extras to `references/`

### 4. Description Format

The `description` field in YAML frontmatter MUST follow this structure:

```
[What it does] + [Use when ...] + [Do NOT use for ...]
```

- **Under 1024 characters**
- **"Use when"** is mandatory — include trigger phrases users would say
- **"Do NOT use for"** is mandatory — prevent overlap with similar skills

## Contributing Process

1. **Fork** this repository
2. **Create a branch**: `git checkout -b add-my-skill`
3. **Add your skill** in the correct category folder
4. **Test**: Ensure `SKILL.md` has valid YAML frontmatter
5. **Submit a PR** with a description of what the skill teaches

## Suggested Skills to Create

These are skills we'd love to see contributed:

- [ ] `scaffold-stellar` — Using the Scaffold Stellar starter
- [ ] `stellar-observatory` — Stellar Observatory dashboard
- [ ] `passkey-wallets` — Smart accounts with passkeys
- [ ] `stellar-defi-gotchas` — Common DeFi mistakes on Stellar
- [ ] `hackathon-guide` — Tips for Stellar hackathons
- [ ] `ya-otter-save` — Fiat savings flow reference
- [ ] `soroban-advanced-patterns` — Factory, proxy, upgradeability
- [ ] `stellar-ecosystem-db` — Ecosystem database integration

## Resources for Skill Authors

- Stellar Docs: https://developers.stellar.org/
- llms.txt (AI-optimized docs): https://developers.stellar.org/llms.txt
- Stellar GitHub: https://github.com/stellar
- StellarSkills reference: https://github.com/ggoldani/stellarskills
- Stellar Dev Skill: https://github.com/stellar/stellar-dev-skill
- Hackathon FAQ: https://github.com/briwylde08/stellar-hackathon-faq
- DeFi Gotchas: https://github.com/kaankacar/stellar-defi-gotchas
- Ecosystem DB: https://github.com/lumenloop/stellar-ecosystem-db
- Ecosystem Resources: https://github.com/stellar/ecosystem-resources/
