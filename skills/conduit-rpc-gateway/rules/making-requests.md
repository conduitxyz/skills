# Making requests

## With the Tempo Wallet CLI (recommended for agents)

Quote first, get the user's approval, then pay pinned to the quote:

```bash
# Quote: price, session deposit, token, and network; nothing is signed or paid
tempo request -t --dry-run --network mainnet --max-spend 1 -X POST \
  --json '{"jsonrpc":"2.0","method":"eth_chainId","params":[],"id":1}' \
  https://mpp.conduit.xyz/zora-mainnet-0

# Pay, after approval
tempo request -t --network mainnet --max-spend 1 --payment-intent session --payment-token USDC.e -X POST \
  --json '{"jsonrpc":"2.0","method":"eth_chainId","params":[],"id":1}' \
  https://mpp.conduit.xyz/zora-mainnet-0
```

- Keep `--max-spend` on every call: it caps the session deposit and its cumulative spend.
- The first paid call opens a session; later calls to any Conduit network reuse it.
- `E_LIMIT_EXCEEDED` means the budget, or the agent key's limit, is too low. Report the required amount and ask before raising it.
- When a wallet or key problem is unclear, run `tempo wallet doctor --format json` and report its failing checks.

See [curl-workflow](curl-workflow.md) for more examples.

## With the mppx CLI

A lightweight client for development and debugging. It handles the full payment flow automatically.

```bash
npx mppx --network mainnet -X POST https://mpp.conduit.xyz/zora-mainnet-0 \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_chainId","params":[],"id":1}'
```

`mppx` keeps the session for reuse. Close it with `npx mppx sessions close <channel-id>` (list with `npx mppx sessions list`) once the user is done.

## With the mppx SDK (tempo.session.manager)

For programmatic access with explicit channel lifecycle control. In mppx ≥ 0.7,
the standalone session lives on `tempo.session.manager(...)` — calling
`tempo.session({ client })` directly no longer returns a fetchable session.

```typescript
import { tempo } from 'mppx/client'
import { privateKeyToAccount } from 'viem/accounts'

const GATEWAY = 'https://mpp.conduit.xyz'

const account = privateKeyToAccount('0x...')

// Create a payment session
// maxDeposit: total locked into escrow (not per-request cost)
// At $0.00005/request, $1 covers ~20,000 requests
const session = tempo.session.manager({
  account,
  maxDeposit: '1',
})

// Make paid RPC calls
const res = await session.fetch(`${GATEWAY}/zora-mainnet-0`, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    jsonrpc: '2.0',
    method: 'eth_chainId',
    params: [],
    id: 1,
  }),
})

const data = await res.json()
console.log(data) // {"jsonrpc":"2.0","result":"0x76adf1","id":1}

// Close the channel when done -- settles on-chain, refunds unused deposit
const receipt = await session.close()
```

## Multiple networks in one session

A single payment session can make calls across different networks:

```typescript
await session.fetch(`${GATEWAY}/tempo`, { ... })           // opens channel
await session.fetch(`${GATEWAY}/zora-mainnet-0`, { ... })  // voucher
await session.fetch(`${GATEWAY}/mode-mainnet-0`, { ... })  // voucher
await session.fetch(`${GATEWAY}/bob-mainnet-0`, { ... })   // voucher

// Close once at the end
await session.close()
```

## Session management (Tempo Wallet CLI)

When using the Tempo Wallet CLI (`tempo request`), payment sessions are managed separately from the request commands. After completing a batch of queries, **ask the user if they want to continue or close the session**.

```bash
# List active sessions
tempo wallet sessions list

# Sync local records with on-chain state
tempo wallet sessions sync

# Preview, then close the Conduit session (settles on-chain, refunds the unused deposit)
tempo wallet sessions close https://mpp.conduit.xyz --cooperative --dry-run
tempo wallet sessions close https://mpp.conduit.xyz --cooperative --yes

# Close only orphaned sessions
tempo wallet sessions close --orphaned --dry-run
```

**Do NOT close sessions without asking the user first.** The user may want to make additional calls using the same session.

## Error handling

```typescript
const res = await session.fetch(`${GATEWAY}/zora-mainnet-0`, { ... })

if (res.status === 404) {
  // Unknown network ID
  // List valid IDs from https://mpp.conduit.xyz/openapi.json
}

// JSON-RPC errors come back as 200 with an error field
const data = await res.json()
if (data.error) {
  console.error(`RPC error: ${data.error.message}`)
}
```
