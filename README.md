# Conduit Skills

Agent Skills for integrating [Conduit](https://conduit.xyz) RPC infrastructure into AI-powered applications.

## Skills

### `skills/conduit-rpc-gateway`

Paid JSON-RPC access to all Conduit chains via the [Machine Payments Protocol](https://mpp.dev/overview). Supports 60+ networks including Tempo, Plume, and Polygon Katana.

- **Auth**: MPP session (pay-as-you-go payment channels via Tempo)
- **Cost**: $0.00005 per JSON-RPC call
- **Setup**: Tempo Wallet or mppx CLI with a funded wallet
- **Entry point**: [`skills/conduit-rpc-gateway/SKILL.md`](skills/conduit-rpc-gateway/SKILL.md)

## Installation

```bash
npx skills add conduitxyz/skills --yes
```

## Official Links

- [Conduit](https://conduit.xyz)
- [Conduit Hub](https://hub.conduit.xyz)
- [Conduit Docs](https://docs.conduit.xyz)
- [MPP overview](https://mpp.dev/overview)

## Specification

These skills follow the [Agent Skills specification](https://agentskills.io/specification). See [spec/agent-skills-spec.md](spec/agent-skills-spec.md) for details.

## License

MIT
