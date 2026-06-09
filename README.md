# Agent Payments (x402)

Batch payments for AI agents across **Base** and additional EVM chains, plus payroll, invoicing, and escrow on Base — settled in USDC via [x402](https://x402.org): no API keys, no accounts. Implements [BPA 1.0](https://docs.spraay.app/bpa/1.0/).

## Install

```bash
npx skills add plagtech/agent-payments-x402
```

## What it does

Send USDC to many wallets in one transaction across Base and additional EVM chains, then run stablecoin payroll, create payment-gated invoices, and lock funds in on-chain escrow on Base — all from an agent. Full endpoint list and examples are in [`SKILL.md`](./SKILL.md).

## Payment

```bash
npm install x402-client
export EVM_PRIVATE_KEY=0x...   # wallet holding USDC on Base
```

Powered by the [Spraay x402 Gateway](https://gateway.spraay.app) · [Docs](https://docs.spraay.app) · [Live](https://live.spraay.app)
