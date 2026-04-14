# Networks

## Supported networks

All networks listed on the [Conduit Hub](https://hub.conduit.xyz) or configured as discoverable are automatically supported. This includes 60+ networks such as Tempo, Plume, and Polygon Katana.

For more on Conduit's RPC infrastructure, see the [Conduit RPC Nodes docs](https://docs.conduit.xyz/rpc-nodes/overview).

You can also discover available networks programmatically via the gateway's [discovery endpoints](discovery.md):

```bash
curl https://mpp.conduit.xyz/openapi.json  # machine-readable OpenAPI document
curl https://mpp.conduit.xyz/llms.txt      # LLM-friendly overview
```

## Route format

```
POST https://mpp.conduit.xyz/:network-id
```

Where `:network-id` is the network's slug identifier. Examples:

| Network            | Route                           |
| ------------------ | ------------------------------- |
| Tempo              | `/tempo`                        |
| Tempo Testnet      | `/tempo-moderato`               |
| Zora Mainnet       | `/zora-mainnet-0`               |
| Mode Mainnet       | `/mode-mainnet-0`               |
| BOB                | `/bob-mainnet-0`                |
| Derive             | `/derive-mainnet-0`             |
| Proof of Play Apex | `/proof-of-play-apex-mainnet-0` |
