---
name: Agent Payments
description: Batch payments for AI agents across Base and additional EVM chains, plus payroll, invoicing, and escrow on Base. Send USDC to many recipients in one transaction, run stablecoin payroll, create payment-gated invoices, and lock funds in on-chain escrow. Pay-per-call via x402 — no API keys, no accounts. Powered by Spraay Protocol.
license: MIT
version: 1.0.0
homepage: https://github.com/plagtech/agent-payments-x402
---

# Agent Payments

Move money on-chain from an agent without writing contract code. This skill wraps the **Spraay x402 Gateway** (`gateway.spraay.app`) — batch disbursement across Base and additional EVM chains, plus payroll, invoicing, and escrow on Base. Every call is paid per-use with the x402 protocol, so there are no API keys, accounts, or subscriptions.

The batch and payroll endpoints implement **Batch Payments for Agents (BPA) 1.0**, an open settlement spec: https://docs.spraay.app/bpa/1.0/

## When to use this skill

Use it when the task involves *sending* or *holding* value:
- Pay many wallets at once (airdrops, rewards, contributor payouts, refunds)
- Run recurring or one-off stablecoin payroll
- Create an invoice that auto-settles to a wallet when paid
- Lock funds in escrow and release them on a condition

Do **not** use it for read-only DeFi data (prices, swaps, wallet analytics) — that's the `defi-intelligence` skill in this repo.

## Setup

Payment is automatic via the x402 client. You need a wallet funded with USDC on Base.

```bash
npm install x402-client
export EVM_PRIVATE_KEY=0x...   # wallet holding USDC on Base
```

```js
import { createClient } from "x402-client";
const client = createClient({
  baseUrl: "https://gateway.spraay.app",
  privateKey: process.env.EVM_PRIVATE_KEY,
});
```

When you hit an endpoint without payment, the gateway returns HTTP 402 with the exact price; the client signs the USDC authorization and retries automatically.

## Endpoints

### Batch payments (BPA 1.0)
| Method | Path | Price | Description |
|---|---|---|---|
| POST | `/api/v1/batch/execute` | $0.02 | Batch-send native or ERC-20 to many recipients in one tx. Chains: Base, Ethereum, Arbitrum, Polygon, BNB, Avalanche, Unichain, Plasma, BOB. |
| POST | `/api/v1/batch/estimate` | $0.001 | Estimate gas + fees before executing a batch. |

### Payroll
| Method | Path | Price | Description |
|---|---|---|---|
| POST | `/api/v1/payroll/execute` | $0.10 | Run a payroll batch — stablecoin payments to a roster. |
| POST | `/api/v1/payroll/estimate` | $0.003 | Estimate payroll batch cost before executing. |
| GET | `/api/v1/payroll/tokens` | $0.002 | Supported payroll tokens (USDC, USDT, DAI, EURC, ...). |

### Invoicing
| Method | Path | Price | Description |
|---|---|---|---|
| POST | `/api/v1/invoice/create` | $0.05 | Create an x402 payment-gated invoice; shareable link, auto-settles to wallet. |
| GET | `/api/v1/invoice/list` | $0.01 | List your invoices with status and payment history. |
| GET | `/api/v1/invoice/:id` | $0.001 | Fetch invoice details and payment status. |

### Escrow
| Method | Path | Price | Description |
|---|---|---|---|
| POST | `/api/v1/escrow/create` | $0.10 | Create an on-chain escrow between two parties. |
| POST | `/api/v1/escrow/fund` | $0.02 | Fund an existing escrow with USDC or a supported token. |
| POST | `/api/v1/escrow/release` | $0.08 | Release escrow funds after conditions are met. |
| POST | `/api/v1/escrow/cancel` | $0.02 | Cancel an escrow before funding or by mutual agreement. |
| GET | `/api/v1/escrow/list` | $0.02 | List active and historical escrows. |
| GET | `/api/v1/escrow/:id` | $0.001 | Fetch escrow details and status. |

## Examples

Batch-send USDC to 50 wallets on Base:
```js
const tx = await client.post("/api/v1/batch/execute", {
  chain: "base",
  token: "USDC",
  recipients: [
    { address: "0xabc...", amount: "10.00" },
    { address: "0xdef...", amount: "25.50" },
    // ...
  ],
});
```

Estimate first, then run payroll:
```js
const quote = await client.post("/api/v1/payroll/estimate", { token: "USDC", roster });
const run = await client.post("/api/v1/payroll/execute", { token: "USDC", roster });
```

Create a payment-gated invoice:
```js
const invoice = await client.post("/api/v1/invoice/create", {
  amount: "250.00", token: "USDC", memo: "Design work — March",
});
// invoice.url is a shareable x402 link that settles to your wallet
```

## Notes
- Settlement is USDC on Base by default. Multi-chain support applies to batch payments only (the chains listed in the batch row above); payroll, invoicing, and escrow settle on Base.
- Spraay batch contract on Base: `0x1646452F98E36A3c9Cfc3eDD8868221E207B5eEC`
- Full catalog: https://docs.spraay.app · Live traffic: https://live.spraay.app
