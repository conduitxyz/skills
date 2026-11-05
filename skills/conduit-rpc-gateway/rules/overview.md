# Overview

## Gateway URL

```
https://mpp.conduit.xyz
```

## Payment

Uses [mppx sessions](https://mpp.dev/guides/pay-as-you-go) (pay-as-you-go payment channels via **Tempo**). Payments are in USD stablecoins on the Tempo blockchain.

- Network: **Tempo mainnet** (chain ID `4217`) only
- Accepted currencies: **USDC.e** (offered first) or **OUSD**. Both have 6 decimals. A channel uses one currency for its whole life.
- Cost per request: **$0.00005**
- Payment channel deposit: capped by `--max-spend` (Tempo CLI) or `maxDeposit` (SDK); the server suggests $0.50
- At $0.00005/request, $1 covers ~20,000 requests

## Payment flow

1. Client sends a JSON-RPC request to `/:network-id`
2. Server returns `402 Payment Required` with one `WWW-Authenticate: Payment` challenge per accepted currency (USDC.e first, then OUSD)
3. Client opens a payment channel on-chain (first request only)
4. Client sends a signed voucher with the request
5. Server verifies and proxies the RPC request
6. Response includes `Payment-Receipt`

After the first request, every subsequent request is just a signed voucher -- no gas, no block confirmation, instant.

When done, settle the session to release unused funds:

- **Tempo Wallet CLI:** Run `tempo wallet sessions close https://mpp.conduit.xyz --cooperative --yes` (preview with `--dry-run` first).
- **mppx CLI:** Run `npx mppx sessions close <channel-id>`.
- **SDK:** Call `session.close()`.

The server keeps the owed amount, and the unused deposit is refunded. **Always ask the user before closing** -- they may want to make additional queries using the same session.

## Discovery

The gateway serves built-in discovery endpoints at `/openapi.json` and `/llms.txt`. See [discovery](discovery.md) for details.

## Supported methods

The proxy forwards any valid JSON-RPC request. All standard EVM methods are supported:

| Method                      | Description                      |
| --------------------------- | -------------------------------- |
| `eth_chainId`               | Get the chain ID                 |
| `eth_blockNumber`           | Get the latest block number      |
| `eth_getBalance`            | Get an address's ETH balance     |
| `eth_getTransactionReceipt` | Get a transaction receipt        |
| `eth_call`                  | Call a contract read function    |
| `eth_getBlockByNumber`      | Get a block by number            |
| `eth_getLogs`               | Get event logs matching a filter |
| `eth_estimateGas`           | Estimate gas for a transaction   |
| `eth_getCode`               | Get contract bytecode            |
| `eth_getTransactionCount`   | Get an address's nonce           |
