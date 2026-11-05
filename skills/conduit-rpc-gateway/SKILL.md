---
name: conduit-rpc-gateway
description: >
  Lets agents make paid JSON-RPC calls to networks on Conduit Nodes. Uses MPP sessions on Tempo.
  Use for ANY EVM JSON-RPC task (reading data, checking balances, or sending transactions).
  CRITICAL LIMITATION: Before the first payment that opens a new channel, you MUST quote it
  (`tempo request --dry-run`) and OBTAIN EXPLICIT HUMAN APPROVAL of the per-call price and the
  session deposit. Once the human approves, you may make all subsequent RPC calls autonomously
  within the approved `--max-spend` budget until the task is complete.
license: MIT
compatibility: Requires the Tempo CLI (recommended) or Node.js (npx mppx), and a Tempo mainnet wallet funded with USDC.e or OUSD. Works across Claude Code, Amp, Codex, Claude.ai, and API.
metadata:
  author: conduitxyz
  version: "1.3"
---

# Conduit RPC Gateway

Paid JSON-RPC access to [Conduit](https://www.conduit.xyz) chains using the [Machine Payments Protocol](https://mpp.dev/overview).

All public networks on [Conduit Nodes](https://hub.conduit.xyz) are supported. Each network is a paid route at `/:network-id`. Cost is **$0.00005** per JSON-RPC call, paid in **USDC.e** or **OUSD** on Tempo mainnet through a pay-as-you-go session.

## Gateway URL

```
https://mpp.conduit.xyz
```

The gateway is listed in the Tempo service directory as `conduit` (`tempo wallet services show conduit`).

## Setup (Required)

**BEFORE making any requests**, you MUST determine which client to use. Follow this flow:

1. **Ask the user which client they prefer.** Present this prompt exactly:

> Which client would you like to use for Conduit RPC calls?
>
> 1. **Tempo Wallet** (Recommended) — Tempo's agent wallet CLI with access-key spending limits, cost previews, and session management. No manual key management.
> 2. **mppx** — Lightweight MPP client via `npx`, best for development and debugging. Manages a local key.

**Do NOT skip this prompt. Do NOT pick a client on behalf of the user.** Wait for their explicit choice before proceeding.

2. **Based on the user's choice**, follow the setup instructions in [wallet-bootstrap](rules/wallet-bootstrap.md).

## Use when

- The user asks to query a Conduit chain (Tempo, Plume, Katana, Zora, Mode, BOB, etc.)
- An agent needs to make EVM JSON-RPC calls to a Conduit chain
- Fetching chain data: block numbers, balances, transaction receipts, contract reads
- The user mentions Conduit, Conduit chains, or any Conduit-hosted chain
- The user needs RPC access to a chain without running a node

## Quick start (Tempo Wallet)

```bash
# 1. Quote: shows the price, session deposit, token, and network without paying
tempo request -t --dry-run --network mainnet --max-spend 1 -X POST \
  --json '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' \
  https://mpp.conduit.xyz/zora-mainnet-0

# 2. After the user approves the quote, pay, pinned to what was approved
tempo request -t --network mainnet --max-spend 1 --payment-intent session --payment-token USDC.e -X POST \
  --json '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' \
  https://mpp.conduit.xyz/zora-mainnet-0
```

Later calls within the budget reuse the open session and only sign off-chain vouchers. See [curl-workflow](rules/curl-workflow.md) for more examples.

## Rules

| Rule                                          | Description                                              |
| --------------------------------------------- | -------------------------------------------------------- |
| [wallet-bootstrap](rules/wallet-bootstrap.md) | Wallet setup and client selection (mandatory first step) |
| [overview](rules/overview.md)                 | Gateway overview, payment flow, supported methods        |
| [payment](rules/payment.md)                   | Currencies, sessions, budgets, receipts, error handling  |
| [making-requests](rules/making-requests.md)   | Making paid RPC calls with the Tempo CLI, mppx SDK and CLI |
| [networks](rules/networks.md)                 | Available networks and how to find them                  |
| [discovery](rules/discovery.md)               | Built-in discovery endpoints for agents and tools        |
| [curl-workflow](rules/curl-workflow.md)       | Quick RPC calls from the command line                    |

## Discovery

The gateway serves built-in [discovery endpoints](rules/discovery.md) so agents can find available networks, prices, and accepted currencies without prior knowledge:

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

1. **Quote first (MANDATORY):** With Tempo Wallet, run the same request with `--dry-run`. Show the user the per-call `amount`, the session `deposit` it would lock, the `symbol` (token), and the `network`.
2. **First-Time Authorization (MANDATORY):** STOP and ask the user for explicit permission. Prompt the user: *"Connecting to this network opens a payment session that locks a deposit of <deposit> <symbol> and costs <amount> per call. Do you authorize this, with a total budget of <max-spend>?"*
3. **Wait for Input:** Do NOT pay, and do NOT pass confirmation flags such as `--yes`, until the user explicitly approves. With mppx, do not skip its confirmation prompts.
4. **Execute Protocol Flow:** Once the user approves:
   - **Tempo Wallet:** rerun the request without `--dry-run`, with `--max-spend <approved budget>` and the quoted `--network`, `--payment-intent`, and `--payment-token`.
   - **mppx CLI / SDK:** `mppx` and `tempo.session.manager` handle the `402` automatically: they open the channel, then sign vouchers.
   - **Manual:** Extract the `WWW-Authenticate` header, create a credential (for example with `npx mppx sign`), and retry with the `Payment` credential in the `Authorization` header.
5. **Autonomous Execution & Settlement:** After the initial authorization, you may keep making calls (off-chain vouchers) automatically within the approved budget until the task is complete. When finished, **ask the user** whether to close the session (see below); closing refunds the unused deposit.

### Session Management

When you have completed a batch of RPC queries, you **MUST** follow this flow:

1. **Ask the user** before closing the session. Prompt exactly:
   > "I've completed the queries. Would you like to make more RPC calls, or should I close the payment session?"
2. **If the user wants more queries**, keep making calls. The existing session stays open and is reused.
3. **If the user is done**, close only the Conduit session:
   - **Tempo Wallet:** preview, then confirm (agent runs require `--yes`):
     ```bash
     tempo wallet sessions close https://mpp.conduit.xyz --cooperative --dry-run
     tempo wallet sessions close https://mpp.conduit.xyz --cooperative --yes
     ```
   - **mppx CLI:** `npx mppx sessions list`, then `npx mppx sessions close <channel-id>`.
   - **SDK:** `await session.close()`.
4. You can inspect sessions at any time with `tempo wallet sessions list` or `npx mppx sessions list`.

**Do NOT close the session without asking the user first.** Open sessions lock deposited funds in escrow; closing releases the unused balance back to the wallet. Prefer closing the Conduit session by origin over `--all`, which closes every service's sessions.

### `Payment network mismatch`

The wallet is on Tempo testnet. The gateway only accepts payment on Tempo mainnet (chain ID `4217`). Pass `--network mainnet` to `tempo request`, or `--network mainnet` to `npx mppx`.

### `E_LIMIT_EXCEEDED`

The payment or session deposit is over the `--max-spend` budget (default 10) or the agent key's spending limit. Report the required amount and ask the user before rerunning with a higher `--max-spend`, or before approving a `tempo wallet keys update`.

### 404 Not Found

The path didn't match any network ID. List the current network IDs from `/openapi.json` (see [discovery](rules/discovery.md)) or browse [Conduit Nodes](https://hub.conduit.xyz). Networks added or removed on Conduit can take up to 6 hours to appear on or disappear from the gateway.

## Resources

- [Conduit](https://conduit.xyz)
- [Conduit Docs](https://docs.conduit.xyz)
- [Conduit Nodes](https://hub.conduit.xyz)
- [Tempo Wallet CLI](https://tempo.xyz/developers/docs/cli/wallet)
- [tempo request](https://tempo.xyz/developers/docs/cli/request)
- [Give an agent a wallet](https://tempo.xyz/developers/docs/accounts/automations/give-an-agent-a-wallet)
- [MPP overview](https://mpp.dev/overview)
- [MPP pay-as-you-go guide](https://mpp.dev/guides/pay-as-you-go)
