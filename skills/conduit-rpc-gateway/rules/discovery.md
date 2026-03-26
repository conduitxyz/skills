# Discovery

The gateway serves built-in discovery endpoints that describe all available networks, routes, and pricing. Agents and CLI tools use these to understand what the gateway offers without prior knowledge.

## Endpoints

| Endpoint                        | Description                                                                                    |
| ------------------------------- | ---------------------------------------------------------------------------------------------- |
| `GET /discover`                 | Lists all services. Returns JSON by default, markdown for AI user agents and terminal clients. |
| `GET /discover/{network-id}`    | Details for a single network, including routes and pricing.                                    |
| `GET /discover/{network-id}.md` | Markdown description of a single network.                                                      |
| `GET /discover/all`             | All networks with full route details.                                                          |
| `GET /discover/all.md`          | Markdown listing of all networks and routes.                                                   |
| `GET /llms.txt`                 | LLM-friendly overview of the gateway and its services.                                         |

## Content negotiation

The gateway returns **markdown** instead of JSON when the request comes from a known AI user agent (e.g. ChatGPT-User, ClaudeBot, PerplexityBot) or a terminal client (e.g. curl, HTTPie, mppx). Otherwise it returns JSON.

## Examples

### List all networks

```bash
curl https://mpp.conduit.xyz/discover
```

### Get details for a specific network

```bash
curl https://mpp.conduit.xyz/discover/zora-mainnet-0
```

### Get LLM-friendly overview

```bash
curl https://mpp.conduit.xyz/llms.txt
```

### Get full markdown listing

```bash
curl https://mpp.conduit.xyz/discover/all.md
```

## Using discovery programmatically

```typescript
// Fetch all available networks as JSON
const res = await fetch("https://mpp.conduit.xyz/discover", {
  headers: { Accept: "application/json" },
});
const services = await res.json();

// Each service has id, title, description, and routes with pricing
for (const service of services) {
  console.log(`${service.id}: ${service.title}`);
  for (const route of service.routes) {
    console.log(`  ${route.pattern} — $${route.payment?.amount / 1e6}/request`);
  }
}
```
