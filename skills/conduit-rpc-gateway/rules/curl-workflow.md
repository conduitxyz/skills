# curl workflow

Quick reference for making RPC calls from the command line.

## Option 1: Tempo Wallet (Recommended)

The [Tempo Wallet](https://wallet.tempo.xyz) is a managed MPP client with built-in spend controls and service discovery.

### Setup (one-time)

```bash
curl -fsSL https://tempo.xyz/install | bash
tempo wallet login
```

### Make a paid RPC call

```bash
tempo request -X POST https://mpp.conduit.xyz/zora-mainnet-0 \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_chainId","params":[],"id":1}'
```

## Option 2: mppx CLI

The `mppx` CLI is a lightweight MPP client bundled with the `mppx` package.

### Setup (one-time)

```bash
npx mppx account create
```

### Make a paid RPC call

```bash
npx mppx -X POST https://mpp.conduit.xyz/zora-mainnet-0 \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_chainId","params":[],"id":1}'
```

## Common RPC calls

These examples use `mppx`. Replace `npx mppx` with `tempo request` if using Tempo Wallet.

### Get chain ID

```bash
npx mppx -X POST https://mpp.conduit.xyz/zora-mainnet-0 \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_chainId","params":[],"id":1}'
```

### Get latest block number

```bash
npx mppx -X POST https://mpp.conduit.xyz/zora-mainnet-0 \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}'
```

### Get ETH balance

```bash
npx mppx -X POST https://mpp.conduit.xyz/zora-mainnet-0 \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_getBalance","params":["0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045","latest"],"id":1}'
```

### Call a contract (read-only)

```bash
npx mppx -X POST https://mpp.conduit.xyz/zora-mainnet-0 \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_call","params":[{"to":"0x...","data":"0x..."},"latest"],"id":1}'
```

## Different networks

Change the network ID in the URL:

```bash
# Mode Mainnet
npx mppx -X POST https://mpp.conduit.xyz/mode-mainnet-0 \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}'

# BOB
npx mppx -X POST https://mpp.conduit.xyz/bob-mainnet-0 \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_chainId","params":[],"id":1}'
```

Browse available network IDs on the [Conduit Hub](https://hub.conduit.xyz).
