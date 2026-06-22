# Payment

The `mppx/client` library handles 402 Payment Required flows automatically -- you don't need to manually parse challenges, create credentials, or retry requests. See [making-requests](making-requests.md) for full setup.

## Payment method

| Method | Description | Requirements |
|--------|-------------|-------------|
| **Tempo** | On-chain USDC payment via payment channels | EVM wallet funded with USDC |

## New Sessions (mppx ≥ 0.7)

The Conduit gateway runs the current TIP-1034 Sessions implementation. On the
client, the API to reach for is **`tempo.session.manager(...)`**:

| Client API | Use it when |
| --- | --- |
| `tempo.session.manager({ account, maxDeposit })` | You want a standalone session with direct lifecycle control: `.fetch()`, `.topUp()`, `.close()`, and the running `.cumulative` spend. **This is what the examples below use.** |
| `tempo.session({ account, maxDeposit })` | You only want to register the Sessions method inside `Mppx.create({ methods: [...] })` to get a payment-aware `fetch`. It does **not** return an object with `.fetch()`/`.close()`. |
| `tempo({ account, maxDeposit })` | A fetch wrapper for `Mppx.create` that handles both one-time charges and Sessions. |

What changed vs. legacy sessions:

- Calling `tempo.session({ client })` directly no longer yields a fetchable session — use `tempo.session.manager(...)` instead.
- Settlement is **server-owned and automatic**: the gateway settles accepted vouchers on-chain as usage accumulates. The client still must `close()` to reclaim the unspent deposit (channels do not close themselves).
- `session.fetch()` returns a `PaymentResponse` that carries the parsed receipt and running totals (`res.receipt`, `res.cumulative`, `res.channelId`).

## How it works

`tempo.session.manager` opens a payment channel on the first request and sends off-chain vouchers on subsequent requests. Only two on-chain transactions are needed from the client: open and close.

```typescript
import { tempo } from "mppx/client";
import { privateKeyToAccount } from "viem/accounts";

const account = privateKeyToAccount(process.env.PRIVATE_KEY as `0x${string}`);

// maxDeposit: total locked into escrow (not per-request cost)
// At $0.00005/request, 1 USDC covers ~20,000 requests
const session = tempo.session.manager({
  account,
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

After a successful payment, the response includes a `Payment-Receipt` header.
With `tempo.session.manager`, `session.fetch()` already returns a
`PaymentResponse` with the parsed receipt and running totals attached:

```typescript
const res = await session.fetch("https://mpp.conduit.xyz/zora-mainnet-0", {
  /* ... */
});

console.log("Channel:", res.channelId);
console.log("Cumulative spend (raw units):", res.cumulative);
console.log("Receipt:", res.receipt); // parsed session receipt, or null
```

For a plain `Response` from any other fetch path, parse the header directly:

```typescript
import { Receipt } from "mppx";

const receipt = Receipt.fromResponse(response);
// `reference` is the channel ID, not a transaction hash. The on-chain
// settlement tx hash is on the receipt returned by `session.close()`
// (`receipt.txHash`).
console.log("Channel:", receipt?.reference);
```

## Payment errors

If a payment fails, the gateway returns 402 with the original challenge. Common issues:

- **Insufficient USDC balance** -- fund the wallet with more USDC
- **Channel exhausted** -- the cumulative voucher amount exceeds the `maxDeposit`. Add headroom without closing the channel by calling `session.topUp('1')`, or close the session and open a new one with a larger deposit.
