# Networks

## Supported networks

All public networks on [Conduit Nodes](https://hub.conduit.xyz) are automatically supported, including Tempo, Plume, and Katana. The gateway reloads its network list from Conduit every 6 hours, so a newly added or removed network can take that long to appear or disappear.

Always take network IDs from `/openapi.json` rather than guessing: each path is `/<network-id>/`.

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

| Network                | Route             |
| ---------------------- | ----------------- |
| Tempo                  | `/tempo`          |
| Tempo Moderato Testnet | `/tempo-moderato` |
| Katana                 | `/katana`         |
| Plume                  | `/plume-mainnet-1` |
| Zora Mainnet           | `/zora-mainnet-0` |
| Mode Mainnet           | `/mode-mainnet-0` |
| BOB Mainnet            | `/bob-mainnet-0`  |

Payment always settles on Tempo mainnet, including for testnet routes such as `/tempo-moderato`.

### List network IDs

```bash
curl -s https://mpp.conduit.xyz/openapi.json | jq -r '.paths | keys[] | ltrimstr("/") | rtrimstr("/")'
```
