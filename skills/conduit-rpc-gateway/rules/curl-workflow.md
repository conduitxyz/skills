# curl workflow

Quick reference for making RPC calls from the command line.

## Option 1: Tempo Wallet (Recommended)

The [Tempo Wallet CLI](https://tempo.xyz/developers/docs/cli/wallet) is Tempo's agent wallet: `tempo wallet` manages the login, agent keys, balances, and sessions, and [`tempo request`](https://tempo.xyz/developers/docs/cli/request) is a curl-compatible client that pays `402` challenges.

### Setup (one-time)

See [wallet-bootstrap](wallet-bootstrap.md). In short:

```bash
curl -fsSL https://tempo.xyz/install | bash
"$HOME/.tempo/bin/tempo" wallet login --network mainnet
"$HOME/.tempo/bin/tempo" wallet whoami --network mainnet
```

### Quote, then pay

Always quote first. `--dry-run` reads the payment terms without signing or paying:

```bash
tempo request -t --dry-run --network mainnet --max-spend 1 -X POST \
  --json '{"jsonrpc":"2.0","method":"eth_chainId","params":[],"id":1}' \
  https://mpp.conduit.xyz/zora-mainnet-0
```

Show the user the quoted `amount` (per call), `deposit` (locked by a new session), `symbol`, and `network`. After they approve, run the same request without `--dry-run`, pinned to what was approved:

```bash
tempo request -t --network mainnet --max-spend 1 --payment-intent session --payment-token USDC.e -X POST \
  --json '{"jsonrpc":"2.0","method":"eth_chainId","params":[],"id":1}' \
  https://mpp.conduit.xyz/zora-mainnet-0
```

- `--max-spend` caps the session deposit and its cumulative spend. Without it, `tempo request` uses the saved `max-spend` or 10.
- `--payment-token` and `--payment-intent` make the request refuse a challenge on another token or intent. Use the quote's `symbol` (`USDC.e` or `OUSD`).
- The first paid call opens a session; later calls to any Conduit network reuse it and only sign vouchers. A successful call prints a short receipt on stderr; add `--write-meta meta.json` to save it.

### Session management

Each new session locks a deposit in escrow. After completing a batch of queries, **ask the user** whether they want to continue or close the session.

```bash
# List active sessions
tempo wallet sessions list

# Sync local session state with on-chain state
tempo wallet sessions sync

# Preview, then close the Conduit session (agent runs need --yes)
tempo wallet sessions close https://mpp.conduit.xyz --cooperative --dry-run
tempo wallet sessions close https://mpp.conduit.xyz --cooperative --yes

# Without --cooperative, a close starts a grace period; withdraw afterwards
tempo wallet sessions close --finalize --yes

# Close only orphaned sessions (counterparty unreachable)
tempo wallet sessions close --orphaned --dry-run
```

**Do NOT close sessions without asking the user first.** Closing releases unused funds back to the wallet, but the user may want to make additional calls. `--all` closes every service's sessions, not just Conduit's.

## Option 2: mppx CLI

The `mppx` CLI is a lightweight MPP client bundled with the `mppx` package, designed for development and debugging.

### Setup (one-time)

```bash
npx mppx account create
```

### Make a paid RPC call

```bash
npx mppx --network mainnet -X POST https://mpp.conduit.xyz/zora-mainnet-0 \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_chainId","params":[],"id":1}'
```

`mppx` keeps the session open for reuse. Close it when the user is done:

```bash
npx mppx sessions list
npx mppx sessions close <channel-id>
```

## Common RPC calls

These examples use `tempo request` after the session is approved. Replace `tempo request ... --json <body>` with `npx mppx --network mainnet -X POST -H "Content-Type: application/json" -d <body>` if using mppx.

### Get chain ID

```bash
tempo request -t --network mainnet --max-spend 1 \
  --json '{"jsonrpc":"2.0","method":"eth_chainId","params":[],"id":1}' \
  https://mpp.conduit.xyz/zora-mainnet-0
```

### Get latest block number

```bash
tempo request -t --network mainnet --max-spend 1 \
  --json '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' \
  https://mpp.conduit.xyz/zora-mainnet-0
```

### Get ETH balance

```bash
tempo request -t --network mainnet --max-spend 1 \
  --json '{"jsonrpc":"2.0","method":"eth_getBalance","params":["0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045","latest"],"id":1}' \
  https://mpp.conduit.xyz/zora-mainnet-0
```

### Call a contract (read-only)

```bash
tempo request -t --network mainnet --max-spend 1 \
  --json '{"jsonrpc":"2.0","method":"eth_call","params":[{"to":"0x...","data":"0x..."},"latest"],"id":1}' \
  https://mpp.conduit.xyz/zora-mainnet-0
```

## Different networks

Change the network ID in the URL. The same session pays for all of them:

```bash
# Tempo
tempo request -t --network mainnet --max-spend 1 \
  --json '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' \
  https://mpp.conduit.xyz/tempo

# Mode Mainnet
tempo request -t --network mainnet --max-spend 1 \
  --json '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' \
  https://mpp.conduit.xyz/mode-mainnet-0

# BOB
tempo request -t --network mainnet --max-spend 1 \
  --json '{"jsonrpc":"2.0","method":"eth_chainId","params":[],"id":1}' \
  https://mpp.conduit.xyz/bob-mainnet-0
```

`--network mainnet` selects the Tempo network the payment settles on, not the chain you are querying. List network IDs from [discovery](discovery.md).
