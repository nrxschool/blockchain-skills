---
name: mcp-servers
description: MCP servers for Stellar — Model Context Protocol integrations, XDR decoder, AI-assisted development. Use when setting up AI agent tooling for Stellar. Do NOT use for general MCP development or non-Stellar MCP servers.
metadata:
  author: nearx
  version: '1.0.0'
  source: 'Stellar ecosystem repos'
---

# MCP Servers for Stellar

> Model Context Protocol (MCP) servers give AI agents tools to interact with the Stellar network directly.

## Available MCP Servers

### Stellar MCP Server
Full-featured MCP server for Stellar operations:

```bash
# Install and configure
npx @kalepail/stellar-mcp-server
```

GitHub: https://github.com/kalepail/stellar-mcp-server

**Capabilities:**
- Account management
- Transaction building and submission
- Contract deployment and invocation
- Network queries

### XDR MCP (Decoder/Encoder)
Decode and encode Stellar XDR data:

```bash
npx @stellar-experimental/mcp-stellar-xdr
```

GitHub: https://github.com/stellar-experimental/mcp-stellar-xdr

### x402 MCP Demo
MCP server demonstrating x402 payment-gated tools:

GitHub: https://github.com/jamesbachini/x402-mcp-stellar

## Setting Up in Claude Desktop

```json
{
  "mcpServers": {
    "stellar": {
      "command": "npx",
      "args": ["@kalepail/stellar-mcp-server"],
      "env": {
        "STELLAR_NETWORK": "testnet",
        "STELLAR_SECRET_KEY": "S..."
      }
    },
    "stellar-xdr": {
      "command": "npx",
      "args": ["@stellar-experimental/mcp-stellar-xdr"]
    }
  }
}
```

## Setting Up in Cursor

Add to `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "stellar": {
      "command": "npx",
      "args": ["@kalepail/stellar-mcp-server"],
      "env": {
        "STELLAR_NETWORK": "testnet"
      }
    }
  }
}
```

## AI-Assisted Development Resources

### Stellar Dev Skills
Official AI skill for Stellar development:
- GitHub: https://github.com/stellar/stellar-dev-skill
- Install: `npx skills add https://github.com/stellar/stellar-dev-skill`

### OpenZeppelin Skills (Stellar)
Smart contract security skills:
- GitHub: https://github.com/OpenZeppelin/openzeppelin-skills
- Stellar support: https://www.openzeppelin.com/networks/stellar

### LLMs.txt
Stellar documentation optimized for AI ingestion:
- https://developers.stellar.org/llms.txt

### Smart Account Kit
Passkey-based smart accounts for Stellar:
- GitHub: https://github.com/kalepail/smart-account-kit

## AI Integration Reference Implementations

| Project | Description | GitHub |
|---------|------------|--------|
| AI Freighter | AI-assisted wallet integration | https://github.com/carstenjacobsen/ai-freighter-integration |
| AI Soroswap | AI-assisted DEX integration | https://github.com/carstenjacobsen/ai-soroswap-integration |
| AI DeFindex | AI-assisted DeFi index | https://github.com/carstenjacobsen/ai-defindex-integration |
| AI Passkeys | AI-assisted passkey auth | https://github.com/carstenjacobsen/ai-passkeys-integration |
| AI Etherfuse | AI-assisted Etherfuse | https://github.com/carstenjacobsen/ai-etherfuse-integration |

## Sponsored Agent Accounts

Create Stellar accounts for AI agents with sponsorship:
- GitHub: https://github.com/oceans404/stellar-sponsored-agent-account
- SKILL.md: https://stellar-sponsored-agent-account.onrender.com/SKILL.md

## Free AI Setup Guide
Complete setup guide for AI development on Stellar:
- https://github.com/kaankacar/stellar-ai-guide-mx/blob/main/Free_AI_Setup.md
