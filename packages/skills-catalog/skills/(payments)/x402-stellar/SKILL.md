---
name: x402-stellar
description: x402 HTTP micropayments on Stellar — pay-per-request APIs using HTTP 402, Soroban token transfers, facilitator pattern. Use when building pay-per-use APIs or machine payments. Do NOT use for general payment flows or EVM x402.
metadata:
  author: nearx
  version: '1.0.0'
  source: 'Stellar Docs + stellarskills + x402-stellar repo'
---

# x402 on Stellar

> Pay-per-request APIs using HTTP **402 Payment Required** on Stellar. Uses Soroban token transfers and authorization entry signing. This is **NOT** identical to x402 on Base/EVM.

## What is x402?

x402 enables machine-to-machine payments over HTTP. An AI agent (or any client) calls a protected API, receives a 402 response with payment requirements, pays via Stellar, and retries to get the resource.

## Official Resources
- x402 on Stellar docs: https://developers.stellar.org/docs/build/apps/x402
- Built on Stellar facilitator: https://developers.stellar.org/docs/build/apps/x402/built-on-stellar
- Stellar x402 repository: https://github.com/stellar/x402-stellar
- Protocol spec: https://x402.org
- Test services: https://xlm402.com
- npm package: https://www.npmjs.com/package/@x402/stellar

## How Stellar x402 Differs from EVM

- **Ledger-based expiration** (not only wall-clock timestamps)
- **Auth entry signing** — client signs authorization entries; facilitator verifies and submits
- **Default asset** is USDC as Soroban token (7 decimals on Stellar)
- **Mainnet** requires an RPC URL from a provider

## Installation

```bash
npm install @x402/stellar @x402/core
```

Optional: `@x402/fetch` for HTTP client wrapper.

## High-Level Flow

1. Client calls a protected HTTP resource
2. Server responds with **402** and payment requirements
3. Client uses `@x402/stellar` to sign auth entries
4. Client retries with the `X-PAYMENT` header
5. **Facilitator** validates and settles on-chain
6. Server returns the resource

## Client Pattern

```typescript
import { x402Client } from "@x402/core/client";
import { createEd25519Signer } from "@x402/stellar";
import { ExactStellarScheme } from "@x402/stellar/exact/client";

const signer = createEd25519Signer(
  process.env.STELLAR_SECRET_KEY!,
  "stellar:testnet"
);

const client = new x402Client()
  .register("stellar:*", new ExactStellarScheme(signer));

// Use with @x402/fetch or your HTTP layer
```

Use `stellar:pubnet` / `stellar:testnet` (CAIP-2 identifiers).

## Server Pattern (Express)

```typescript
import express from "express";

const app = express();

// Protected endpoint
app.get("/api/data", async (req, res) => {
  const payment = req.headers["x-payment"];
  
  if (!payment) {
    return res.status(402).json({
      // Payment requirements
      scheme: "exact",
      network: "stellar:testnet",
      payTo: "GPAYEE...",
      maxAmountRequired: "100000", // 0.01 USDC (7 decimals)
      asset: usdcContractId,
      description: "Access to premium data",
    });
  }
  
  // Verify payment via facilitator
  // ... verification logic
  
  res.json({ data: "premium content" });
});
```

## Demo MCP Server

An MCP server demonstrating x402 payments:
https://github.com/jamesbachini/x402-mcp-stellar

## Stellar MPP SDK

Experimental SDK for machine-to-machine payments on Stellar:
https://github.com/stellar-experimental/stellar-mpp-sdk

## USDC on Stellar

| Network | Issuer |
|---------|--------|
| **Mainnet** | `GA5ZSEJYB37JRC5AVCIA5MOP4RHTM335X2KGX3IHOJAPP5RE34K4KZVN` |
| **Testnet** | `GBBD47IF6LWK7P7MDEVSCWR7DPUWV3NY3DTQEVFL4NAT4AQH3ZLLFLA5` |

## Related Protocol References
- Coinbase x402 docs: https://docs.cdp.coinbase.com/x402/docs/welcome
- x402.org spec: https://www.x402.org/
- Stripe Machine Payments: https://stripe.com/machine-payments
- Stripe MPP docs: https://docs.stripe.com/machine-payments

## Official Documentation
- x402 quickstart: https://developers.stellar.org/docs/build/apps/x402/quickstart-guide
- x402 on Stellar: https://developers.stellar.org/docs/build/apps/x402
- Live demo: https://x402-stellar-491bf9f7e30b.herokuapp.com/
