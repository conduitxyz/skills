# Payment

The `mppx/client` library handles 402 Payment Required flows automatically -- you don't need to manually parse challenges, create credentials, or retry requests. See [making-requests](making-requests.md) for full setup.

## Payment method

| Method | Description | Requirements |
|--------|-------------|-------------|
| **Tempo** | On-chain USDC payment via payment channels | EVM wallet funded with USDC |

## How it works

`tempo.session` opens a payment channel on the first request and sends off-chain vouchers on subsequent requests. Only two on-chain transactions are needed: open and close.

```typescript
import { tempo } from "mppx/client";
import { createClient, http } from "viem";
import { privateKeyToAccount } from "viem/accounts";
import { tempo as tempoChain } from "viem/chains";

const account = privateKeyToAccount(process.env.PRIVATE_KEY as `0x${string}`);
const client = createClient({
  account,
  chain: tempoChain,
  transport: http(),
});

// maxDeposit: total locked into escrow (not per-request cost)
// At $0.00005/request, 1 USDC covers ~20,000 requests
const session = tempo.session({
  client,
  maxDeposit: "1",
});

// First request opens the channel on-chain
const res1 = await session.fetch("https://mpp.conduit.xyz/zora-mainnet-0", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ id: 1, jsonrpc: "2.0", method: "eth_chainId" }),
});

// Subsequent requests use off-chain vouchers (instant, no gas)
const res2 = await session.fetch("https://mpp.conduit.xyz/mode-mainnet-0", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ id: 1, jsonrpc: "2.0", method: "eth_blockNumber" }),
});

// Close the channel -- settles on-chain, refunds unused deposit
const receipt = await session.close();
```

### Closing sessions with the Tempo Wallet CLI

If using the Tempo Wallet CLI instead of the SDK, sessions are managed with `tempo wallet sessions`:

```bash
# List active sessions
tempo wallet sessions list

# Close all sessions (settles on-chain, refunds unused deposits)
tempo wallet sessions close --all

# Close only orphaned sessions (counterparty unreachable)
tempo wallet sessions close --orphaned

# Preview what would be closed without executing
tempo wallet sessions close --all --dry-run
```

**Important:** Before closing sessions, ask the user if they want to make additional queries. Open sessions can be reused for more requests without opening a new channel.

## Payment receipt

After a successful payment, the response includes a `Payment-Receipt` header:

```typescript
import { Receipt } from "mppx";

const receipt = Receipt.fromResponse(response);
console.log("Transaction:", receipt?.reference);
```

## Payment errors

If a payment fails, the gateway returns 402 with the original challenge. Common issues:

- **Insufficient USDC balance** -- fund the wallet with more USDC
- **Channel exhausted** -- the cumulative voucher amount exceeds the `maxDeposit`. Close the session and open a new one with a larger deposit.
