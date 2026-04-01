# Discovery

The gateway serves built-in discovery endpoints that describe all available networks, routes, and pricing. Agents and CLI tools should use these to understand what the gateway offers without prior knowledge.

**Important:** Legacy `/discover*` routes are removed and return `410 Gone`. Always use `/openapi.json` or `/llms.txt` instead.

## Endpoints

| Endpoint            | Description                                                |
| ------------------- | ---------------------------------------------------------- |
| `GET /openapi.json` | OpenAPI discovery document (machine-readable, canonical)   |
| `GET /llms.txt`     | LLM-friendly text overview of the gateway and its services |

## When to use which

- **`/openapi.json`** — Use for programmatic discovery. Returns a standard OpenAPI document describing all available routes, methods, and pricing. Best for agents that need structured data.
- **`/llms.txt`** — Use for a quick human-readable or LLM-readable summary of the gateway. Good starting point for understanding what the gateway offers.

## Examples

### Get the OpenAPI discovery document

```bash
curl https://mpp.conduit.xyz/openapi.json
```

### Get the LLM-friendly overview

```bash
curl https://mpp.conduit.xyz/llms.txt
```

## Using discovery programmatically

```typescript
// Fetch the OpenAPI document
const res = await fetch("https://mpp.conduit.xyz/openapi.json");
const spec = await res.json();

// Inspect available paths and their pricing
for (const [path, methods] of Object.entries(spec.paths)) {
  console.log(`${path}: ${JSON.stringify(methods)}`);
}
```

## Agent workflow

When an agent needs to discover available networks or pricing, it should:

1. Fetch `/openapi.json` for structured route and pricing data
2. Fetch `/llms.txt` for a quick text overview if needed
3. Use the discovered network IDs to make paid RPC calls via `POST /:network-id`
