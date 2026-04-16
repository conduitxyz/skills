# Making requests

## With the mppx CLI

The simplest way to make paid RPC calls. The CLI handles the full payment flow automatically.

```bash
npx mppx -X POST https://mpp.conduit.xyz/zora-mainnet-0 \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_chainId","params":[],"id":1}'
```

## With the mppx SDK (tempo.session)

For programmatic access with explicit channel lifecycle control.

```typescript
import { tempo } from 'mppx/client'
import { createClient, http } from 'viem'
import { privateKeyToAccount } from 'viem/accounts'
import { tempo as tempoChain } from 'viem/chains'

const GATEWAY = 'https://mpp.conduit.xyz'

const account = privateKeyToAccount('0x...')
const client = createClient({
  account,
  chain: tempoChain,
  transport: http(),
})

// Create a payment session
// maxDeposit: total locked into escrow (not per-request cost)
// At $0.00005/request, 1 USDC covers ~20,000 requests
const session = tempo.session({
  client,
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

# Close all sessions (settles on-chain, refunds unused deposits)
tempo wallet sessions close --all

# Close only orphaned sessions
tempo wallet sessions close --orphaned
```

**Do NOT close sessions without asking the user first.** The user may want to make additional calls using the same session.

## Error handling

```typescript
const res = await session.fetch(`${GATEWAY}/zora-mainnet-0`, { ... })

if (res.status === 404) {
  // Unknown network ID
  // Browse https://hub.conduit.xyz for valid IDs
}

// JSON-RPC errors come back as 200 with an error field
const data = await res.json()
if (data.error) {
  console.error(`RPC error: ${data.error.message}`)
}
```
