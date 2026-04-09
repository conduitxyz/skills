---
name: conduit-rpc-gateway
description: Lets agents make paid JSON-RPC calls to network on Conduit Nodes. Uses MPP sessions (pay-as-you-go payment channels via Tempo) — the first request opens a channel on-chain, subsequent requests send off-chain vouchers with no blockchain round-trip. Supports 59+ networks including Zora, Mode, BOB, and more. Use for ANY task involving Conduit Nodes RPC calls — querying chain data, checking balances, reading contracts, fetching block numbers, or any EVM JSON-RPC method. CRITICAL LIMITATION: When encountering a 402 Payment Required to open a new channel, you MUST PAUSE AND OBTAIN EXPLICIT HUMAN APPROVAL FIRST. Once the human approves this initial deposit, you are fully authorized to proceed autonomously with all subsequent RPC calls, payment vouchers, and EVM transactions until the task is complete.
license: MIT
compatibility: Requires Node.js (npx) and a wallet funded with USDC on Tempo. Works across Claude Code, Claude.ai, and API.
metadata:
  author: conduitxyz
  version: "1.0"
---

# Conduit RPC Proxy

Paid JSON-RPC access to [Conduit](https://www.conduit.xyz) rollup networks using the [Machine Payments Protocol](https://mpp.dev/overview).

All networks on the [Conduit Hub](https://hub.conduit.xyz) or configured as discoverable are supported. Each network is a paid route at `/:network-id`. Cost is **$0.00005** per JSON-RPC call.

## Gateway URL

```
https://mpp.conduit.xyz
```

## Setup (Required)

**BEFORE making any requests**, you MUST determine which client to use. Follow this flow:

1. **Ask the user which client they prefer.** Present this prompt exactly:

> Which client would you like to use for Conduit RPC calls?
>
> 1. **Tempo Wallet** (Recommended) — Managed MPP client with built-in spend controls. No manual key management.
> 2. **mppx** — Lightweight MPP client via `npx`. Manages keys internally.

**Do NOT skip this prompt. Do NOT pick a client on behalf of the user.** Wait for their explicit choice before proceeding.

2. **Based on the user's choice**, follow the setup instructions in [wallet-bootstrap](rules/wallet-bootstrap.md).

## Use when

- The user asks to query a Conduit rollup (Zora, Mode, BOB, etc.)
- An agent needs to make EVM JSON-RPC calls to a Conduit chain
- Fetching chain data: block numbers, balances, transaction receipts, contract reads
- The user mentions Conduit, Conduit chains, or any Conduit-hosted rollup
- The user needs RPC access to a rollup without running a node

## Rules

| Rule                                          | Description                                              |
| --------------------------------------------- | -------------------------------------------------------- |
| [wallet-bootstrap](rules/wallet-bootstrap.md) | Wallet setup and client selection (mandatory first step) |
| [overview](rules/overview.md)                 | Gateway overview, payment flow, supported methods        |
| [payment](rules/payment.md)                   | Payment sessions, receipts, error handling               |
| [making-requests](rules/making-requests.md)   | Making paid RPC calls with mppx SDK and CLI              |
| [networks](rules/networks.md)                 | Available networks and how to find them                  |
| [discovery](rules/discovery.md)               | Built-in discovery endpoints for agents and tools        |
| [curl-workflow](rules/curl-workflow.md)       | Quick RPC calls via mppx CLI                             |

## Discovery

The gateway serves built-in [discovery endpoints](rules/discovery.md) so agents can find available networks and pricing without prior knowledge:

```bash
curl https://mpp.conduit.xyz/openapi.json  # OpenAPI discovery document (machine-readable)
curl https://mpp.conduit.xyz/llms.txt      # LLM-friendly overview
```

## API Reference

| Route          | Methods                   | Reference                                                  |
| -------------- | ------------------------- | ---------------------------------------------------------- |
| `/:network-id` | `eth_*` standard JSON-RPC | [references/node-json-rpc.md](references/node-json-rpc.md) |

## Troubleshooting

### 402 Payment Required

A `402 Payment Required` response means a payment must be authorized or a channel opened for this network. You must follow these steps exactly:

1. **First-Time Authorization (MANDATORY):** You MUST STOP and ask the user for explicit permission to initiate the payment. Prompt the user: *"Connecting to this network requires a small payment/deposit. Do you authorize this transaction?"*
2. **Wait for Input:** Do NOT pass auto-confirm flags (such as `-y` or `--yes`) to any CLI commands. Wait for the user to explicitly approve the transaction.
3. **Execute Protocol Flow:** Once the user approves, proceed based on the response headers:
   - **x402**: Extract the `PAYMENT-REQUIRED` header, run `npx @alchemy/x402 pay`, and retry the request with the `Payment-Signature` header.
   - **MPP**: Extract the `WWW-Authenticate` header, create a credential with `mppx`, and retry the request with the `Payment` credential in the `Authorization` header.
   - *(See the payment rule for your chosen protocol for additional details).*
4. **Autonomous Execution & Settlement:** After the initial authorization, you may send subsequent off-chain signatures or vouchers automatically until the task is complete. When finished, ensure you settle the session (e.g., calling `session.close()`) to refund any unused deposits.

### 404 Not Found

The path didn't match any network ID. Browse available networks on the [Conduit Hub](https://hub.conduit.xyz).

## Resources

- [Conduit](https://conduit.xyz)
- [Conduit Docs](https://docs.conduit.xyz)
- [Conduit Hub](https://hub.conduit.xyz)
- [MPP overview](https://mpp.dev/overview)
- [MPP pay-as-you-go guide](https://mpp.dev/guides/pay-as-you-go)
