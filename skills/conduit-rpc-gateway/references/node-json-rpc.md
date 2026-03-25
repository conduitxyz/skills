---
id: conduit-rpc-gateway/references/node-json-rpc.md
name: 'JSON-RPC (EVM)'
description: 'Standard Ethereum JSON-RPC methods via the Conduit RPC Proxy. Paid per-request via MPP sessions.'
tags:
  - conduit
  - rpc
  - json-rpc
  - evm
updated: 2026-03-25
---

# JSON-RPC (EVM)

Standard Ethereum JSON-RPC methods via the Conduit RPC Proxy. All requests are `POST` with JSON-RPC 2.0 bodies.

**Base URL**: `https://mpp.conduit.xyz/:network-id`

**Networks**: All networks on the [Conduit Hub](https://hub.conduit.xyz). Examples: `zora-mainnet-0`, `mode-mainnet-0`, `bob-mainnet-0`.

---

## `eth_blockNumber`

Returns the current block number.

### Parameters

None.

### Request

```bash
npx mppx -X POST https://mpp.conduit.xyz/zora-mainnet-0 \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"eth_blockNumber","params":[]}'
```

### Response

```json
{ "jsonrpc": "2.0", "id": 1, "result": "0x29d8872" }
```

The result is the block number as a hex string.

---

## `eth_chainId`

Returns the chain ID of the network.

### Parameters

None.

### Request

```bash
npx mppx -X POST https://mpp.conduit.xyz/zora-mainnet-0 \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"eth_chainId","params":[]}'
```

### Response

```json
{ "jsonrpc": "2.0", "id": 1, "result": "0x76adf1" }
```

`0x76adf1` = 7777777 (Zora Mainnet).

---

## `eth_getBalance`

Returns the native token balance (ETH) for an address.

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `address` | string | Yes | Wallet address (hex, 20 bytes) |
| `blockTag` | string | No | `"latest"`, `"earliest"`, `"pending"`, `"safe"`, `"finalized"`, or hex block number. Default: `"latest"` |

### Request

```bash
npx mppx -X POST https://mpp.conduit.xyz/zora-mainnet-0 \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "eth_getBalance",
    "params": ["0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045", "latest"]
  }'
```

### Response

```json
{ "jsonrpc": "2.0", "id": 1, "result": "0x78040503cba7a65" }
```

Result is the balance in wei (hex). Divide by 10^18 for ETH.

---

## `eth_call`

Executes a read-only call against a contract (no state changes, no gas cost).

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `transaction.to` | string | Yes | Contract address |
| `transaction.from` | string | No | Caller address |
| `transaction.data` | string | No | ABI-encoded function call (hex) |
| `transaction.value` | string | No | ETH value (hex, usually `"0x0"`) |
| `transaction.gas` | string | No | Gas limit (hex) |
| `blockTag` | string | No | Block tag or hex number. Default: `"latest"` |

### Request

```bash
npx mppx -X POST https://mpp.conduit.xyz/zora-mainnet-0 \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "eth_call",
    "params": [
      {
        "to": "0x...",
        "data": "0x70a08231000000000000000000000000d8da6bf26964af9d7eed9e03e53415d37aa96045"
      },
      "latest"
    ]
  }'
```

### Response

```json
{ "jsonrpc": "2.0", "id": 1, "result": "0x0000000000000000000000000000000000000000000000000000000005f5e100" }
```

Result is the ABI-encoded return data (hex). Decode using the function's return types.

---

## `eth_getLogs`

Returns event logs matching a filter.

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `filter.fromBlock` | string | No | Start block (hex or `"latest"`). Default: `"latest"` |
| `filter.toBlock` | string | No | End block (hex or `"latest"`). Default: `"latest"` |
| `filter.address` | string or string[] | No | Contract address(es) to filter |
| `filter.topics` | array | No | Up to 4 topic filters. `null` means any. |
| `filter.blockHash` | string | No | Filter to a specific block (cannot use with fromBlock/toBlock) |

### Request

```bash
npx mppx -X POST https://mpp.conduit.xyz/zora-mainnet-0 \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "eth_getLogs",
    "params": [{
      "fromBlock": "0x29d8800",
      "toBlock": "0x29d8810",
      "topics": ["0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef"]
    }]
  }'
```

### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `address` | string | Emitting contract address |
| `topics` | string[] | Indexed event parameters |
| `data` | string | Non-indexed event data (hex) |
| `blockNumber` | string | Block number (hex) |
| `transactionHash` | string | Transaction hash |
| `transactionIndex` | string | Transaction index in block (hex) |
| `blockHash` | string | Block hash |
| `logIndex` | string | Log index in block (hex) |
| `removed` | boolean | `true` if removed due to chain reorg |

---

## `eth_getTransactionReceipt`

Returns the receipt of a mined transaction.

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `transactionHash` | string | Yes | Transaction hash (32 bytes, hex) |

### Request

```bash
npx mppx -X POST https://mpp.conduit.xyz/zora-mainnet-0 \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "eth_getTransactionReceipt",
    "params": ["0x...txhash..."]
  }'
```

### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `transactionHash` | string | Transaction hash |
| `blockNumber` | string | Block number (hex) |
| `from` | string | Sender address |
| `to` | string | Recipient address (null for contract creation) |
| `gasUsed` | string | Gas used by this tx (hex) |
| `effectiveGasPrice` | string | Actual gas price paid (hex) |
| `contractAddress` | string | Created contract address (null if not a deployment) |
| `logs` | array | Event logs |
| `status` | string | `"0x1"` (success) or `"0x0"` (failure) |

---

## `eth_getBlockByNumber`

Returns block data by block number.

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `blockTag` | string | Yes | Block number (hex) or `"latest"`, `"earliest"`, `"pending"`, `"safe"`, `"finalized"` |
| `fullTransactions` | boolean | Yes | `true` for full tx objects, `false` for tx hashes only |

### Request

```bash
npx mppx -X POST https://mpp.conduit.xyz/zora-mainnet-0 \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "eth_getBlockByNumber",
    "params": ["latest", false]
  }'
```

### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `number` | string | Block number (hex) |
| `hash` | string | Block hash |
| `parentHash` | string | Parent block hash |
| `timestamp` | string | Block timestamp (hex, Unix seconds) |
| `gasLimit` | string | Gas limit (hex) |
| `gasUsed` | string | Gas used (hex) |
| `baseFeePerGas` | string | Base fee (hex, EIP-1559) |
| `miner` | string | Block producer address |
| `transactions` | array | Transaction hashes (or full objects if `fullTransactions: true`) |

---

## `eth_sendRawTransaction`

Submits a signed transaction to the network.

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `signedTransaction` | string | Yes | Signed transaction data (hex) |

### Request

```bash
npx mppx -X POST https://mpp.conduit.xyz/zora-mainnet-0 \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "eth_sendRawTransaction",
    "params": ["0xf86c...signed_tx_hex..."]
  }'
```

### Response

```json
{ "jsonrpc": "2.0", "id": 1, "result": "0x...transaction_hash..." }
```

Result is the transaction hash. Use `eth_getTransactionReceipt` to check if it was mined.

---

## Other Common Methods

| Method | Description |
|--------|-------------|
| `eth_gasPrice` | Current gas price (hex, wei) |
| `eth_estimateGas` | Estimate gas for a transaction |
| `eth_getTransactionByHash` | Get transaction by hash |
| `eth_getCode` | Get contract bytecode |
| `eth_getStorageAt` | Read contract storage slot |
| `eth_getTransactionCount` | Get nonce for an address |
| `eth_feeHistory` | Fee history for EIP-1559 gas estimation |
| `eth_maxPriorityFeePerGas` | Suggested priority fee |
| `net_version` | Network ID |
| `web3_clientVersion` | Node client version |

---

## Notes

- All quantities are hex-encoded strings (e.g., `"0x10"` = 16).
- Block tags: `"latest"` (most recent), `"finalized"` (finalized by consensus), `"safe"` (safe head), `"earliest"` (genesis), `"pending"` (pending state).
- `eth_getLogs` over large block ranges can be expensive. Use narrow ranges.
- JSON-RPC errors are returned as 200 with an `error` field in the response body (`error.code`, `error.message`).

## Official Docs

- [Conduit RPC Nodes](https://docs.conduit.xyz/rpc-nodes/overview)
- [Making an RPC Request](https://docs.conduit.xyz/rpc-nodes/getting-started/make-rpc-request)
