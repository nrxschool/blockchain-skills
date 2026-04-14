# AGENTS.md

Guidance for AI Coding Agents when working with code in this repository.

## Repository

**Nearx Skills** — A curated collection of AI agent skills focused on the **Stellar Network** and blockchain development. Maintained by the Nearx developer community.

## Architecture

**Nx monorepo** with the following packages:

- **`packages/skills-catalog`** — Skill definitions in `skills/{(category)}/{skill-name}/SKILL.md` with YAML frontmatter. `src/generate-registry.ts` produces `skills-registry.json`.
- **`packages/cli`** — The user-facing CLI installer for skills. Dual-mode: interactive TUI and non-interactive CLI.
- **`libs/core`** — Shared types and constants.
- **`tools/skill-plugin`** — Nx generator for scaffolding new skills.

## Commands

```bash
# Setup
npm ci && npm run build

# Development
npm run start:dev:cli          # Run CLI interactively
SKILLS_CDN_REF=main npm run dev -- install  # Test CLI against local registry

# Build & Test
npm run build                  # Build all packages
npm run test                   # Run all tests
npm run generate:registry      # Regenerate skills-registry.json

# Quality
npm run lint                   # ESLint
npm run format:check           # Prettier

# Create new skill
nx g @nearx/skill-plugin:skill {name} --category={cat}
```

## Skill Structure

```
skills/(category)/{skill-name}/
├── SKILL.md          # Required — YAML frontmatter + markdown body
├── scripts/          # Optional executable scripts
├── templates/        # Optional file templates
└── references/       # Optional on-demand documentation
```

### Skill Categories (Stellar-focused)
- `(stellar-core)` — Accounts, assets, operations, fees
- `(soroban)` — Smart contracts, security, testing
- `(apis)` — Stellar RPC, Horizon
- `(frontend)` — Wallets, browser SDK
- `(payments)` — x402, SEPs, anchors
- `(tools)` — CLI, SDKs
- `(defi)` — DEX, AMM
- `(ai-agents)` — MCP servers, AI integrations
- `(security)` — Contract security, OpenZeppelin
- `(community)` — Contributing guides, learning

### Skill Quality Standards
- **No fluff** — Agents don't need marketing copy
- **Dense & factual** — Prioritize code snippets and architectural rules
- **Current & runnable** — Ensure code works with latest SDK versions
- **Include Common Errors** — Map error codes to fixes
- **Under 500 lines** — Keep focused

## Code Conventions
- **ESM-only** throughout
- **TypeScript strict mode**
- **Prettier**: no semicolons, single quotes, trailing commas, 120 char width
- **Node ≥ 24** (monorepo)

## Workflow Orchestration

### 1. Plan Mode Default
- Enter plan mode for non-trivial tasks
- Use subagents for research and parallel analysis

### 2. Verification Before Done
- Never mark a task complete without proving it works
- Run the full test suite before considering work done

### 3. Simplicity First
- Make every change as simple as possible
- Find root causes, no temporary workarounds
