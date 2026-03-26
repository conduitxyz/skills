# Overview

## Gateway URL

```
https://mpp.conduit.xyz
```

## Payment

Uses [mppx sessions](https://mpp.dev/guides/pay-as-you-go) (pay-as-you-go payment channels via **Tempo**). Payments are in **USDC** on the Tempo blockchain.

- Cost per request: **$0.00005**
- Payment channel deposit: configurable via `maxDeposit` (default: 1 USDC)
- At $0.00005/request, 1 USDC covers ~20,000 requests

## Payment flow

1. Client sends a JSON-RPC request to `/:network-id`
2. Server returns `402 Payment Required` with `WWW-Authenticate: Payment`
3. Client opens a payment channel on-chain (first request only)
4. Client sends a signed voucher with the request
5. Server verifies and proxies the RPC request
6. Response includes `Payment-Receipt`

After the first request, every subsequent request is just a signed voucher -- no gas, no block confirmation, instant.

When done, call `session.close()` to settle on-chain. The server keeps the owed amount, and the unused deposit is refunded.

## Discovery

The gateway serves built-in discovery endpoints at `/discover` and `/llms.txt`. See [discovery](discovery.md) for details.

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
