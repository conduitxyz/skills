# Wallet Setup

**This is the mandatory entry point for paid RPC requests.** No data can be fetched until setup is complete. If the user asks a blockchain question and no client is configured, redirect them here first.

## Step 1: Choose a Client (Hard Requirement)

You MUST ask the user which client they want to use. Present this prompt exactly:

> Which client would you like to use for Conduit RPC calls?
>
> 1. **Tempo Wallet** (Recommended) — Managed MPP client with built-in spend controls. No manual key management.
> 2. **mppx** — Lightweight MPP client via `npx`. Manages keys internally.

**Do NOT skip this prompt. Do NOT pick a client on behalf of the user.** Wait for their explicit choice before proceeding.

---

## Step 2: Set Up the Client

### If the user chose Tempo Wallet

```bash
curl -fsSL https://tempo.xyz/install | bash
tempo wallet login
```

Once logged in, use `tempo request` to make paid calls. Proceed to [making-requests](making-requests.md).

### If the user chose mppx

```bash
npx mppx account create
```

Then use `npx mppx` to make paid calls. Proceed to [making-requests](making-requests.md) or [curl-workflow](curl-workflow.md).

---

## Step 3: Fund the Wallet

The wallet needs USDC on Tempo to pay for RPC calls. At $0.00005 per request, 1 USDC covers ~20,000 RPC calls.

---

## Programmatic Usage (SDK)

For building applications, use `viem` for wallet management and `mppx` for payments.

```typescript
import { tempo } from 'mppx/client'
import { createClient, http } from 'viem'
import { privateKeyToAccount } from 'viem/accounts'
import { tempo as tempoChain } from 'viem/chains'

const account = privateKeyToAccount(process.env.PRIVATE_KEY as `0x${string}`)
const client = createClient({
  account,
  chain: tempoChain,
  transport: http(),
})

const session = tempo.session({
  client,
  maxDeposit: '1',
})
```

> **Important:** Never hardcode private keys in source files. Always read from an environment variable or secure store.

See [making-requests](making-requests.md) for full usage examples.
