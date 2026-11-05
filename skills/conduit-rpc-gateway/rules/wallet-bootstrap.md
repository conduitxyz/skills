# Wallet Setup

**This is the mandatory entry point for paid RPC requests.** No data can be fetched until setup is complete. If the user asks a blockchain question and no client is configured, redirect them here first.

## Step 1: Choose a Client (Hard Requirement)

You MUST ask the user which client they want to use. Present this prompt exactly:

> Which client would you like to use for Conduit RPC calls?
>
> 1. **Tempo Wallet** (Recommended) — Tempo's agent wallet CLI with access-key spending limits, cost previews, and session management. No manual key management.
> 2. **mppx** — Lightweight MPP client via `npx`, best for development and debugging. Manages a local key.

**Do NOT skip this prompt. Do NOT pick a client on behalf of the user.** Wait for their explicit choice before proceeding.

---

## Step 2: Set Up the Client

### If the user chose Tempo Wallet

Tempo's own setup is the source of truth. The quickest path is to follow Tempo's agent setup skill:

```bash
curl -fsSL https://tempo.xyz/skills/wallet/SKILL.md
```

Fetch it with `curl` (a web fetch tool may summarize away the commands) and follow it. It installs the CLI and Tempo's `wallet` skill. In short:

1. **Install:** `curl -fsSL https://tempo.xyz/install | bash`. Call the CLI by its absolute path, `"$HOME/.tempo/bin/tempo"`, instead of changing `PATH`.
2. **Log in:** `"$HOME/.tempo/bin/tempo" wallet login --network mainnet --format json` returns a `verification_url` and `user_code`. Send both to the user, ask them to confirm the code matches and approve in the browser, then run the returned `_next` command (`tempo wallet login status ...`) to finish. If the user is on another device than the CLI host, add `--no-browser`. Do not retry login without the user.
3. **Check readiness:** `"$HOME/.tempo/bin/tempo" wallet whoami --network mainnet`. `ready: true` means the wallet can pay.

The login creates an **agent key** for this machine: the agent signs payments with it, never with the wallet's own key. The key has spending limits the wallet owner sets and can revoke (`tempo wallet keys list`, `tempo wallet keys revoke <ACCESS_KEY> --dry-run`).

The Conduit gateway charges on **Tempo mainnet** (chain ID `4217`). Pass `--network mainnet` if the wallet's saved network is testnet.

If `whoami` or a quote reports an expired key, run `tempo wallet refresh --network mainnet`. Like login, it prints a code and URL for the user to approve.

Proceed to [making-requests](making-requests.md).

### If the user chose mppx

```bash
npx mppx account create
npx mppx account view
```

`account create` stores a new local key and `account view` prints its address to fund. Then use `npx mppx --network mainnet` to make paid calls. Proceed to [making-requests](making-requests.md) or [curl-workflow](curl-workflow.md).

---

## Step 3: Fund the Wallet

The wallet needs **USDC.e** or **OUSD** on Tempo mainnet. Both are live mainnet stablecoins. At $0.00005 per request, $1 covers ~20,000 RPC calls; the gateway suggests a $0.50 session deposit.

- **Tempo Wallet:** check balances with `tempo wallet balance --network mainnet`, and add funds with `tempo wallet fund --network mainnet`. If the user is on another device, add `--no-browser --format json` and hand them the URL. Ask the user to tell you when funding is done, then check the balance again before paying.
- **mppx:** send USDC.e or OUSD on Tempo mainnet to the address from `npx mppx account view`.

The gateway offers USDC.e first, then OUSD. Fund whichever the user prefers; with only OUSD, pay with `--payment-token OUSD` (Tempo Wallet) or `--currency 0x20c0000000000000000000006a37DA5C996874BE` (mppx). See [payment](payment.md).

---

## Programmatic Usage (SDK)

For building applications, use `viem` for accounts and `mppx` for payments.

> **Sessions (mppx ≥ 0.7):** use `tempo.session.manager(...)` for direct
> lifecycle control (`.fetch()`, `.topUp()`, `.close()`, `.cumulative`).
> `tempo.session()` on its own is only a method you register inside
> `Mppx.create({ methods: [...] })`. See [payment](payment.md) for details.

```typescript
import { tempo } from 'mppx/client'
import { privateKeyToAccount } from 'viem/accounts'

const account = privateKeyToAccount(process.env.PRIVATE_KEY as `0x${string}`)

// The manager opens a payment channel on the first request and signs
// off-chain vouchers afterward. The server settles accepted usage on-chain
// automatically; you only call close() to reclaim the unspent deposit.
const session = tempo.session.manager({
  account,
  maxDeposit: '1',
})
```

For an autonomous agent, prefer a dedicated **agent key** with a budget, expiry, and scopes over the account's own key, so a misbehaving agent can only spend within those bounds. See Tempo's [Give an agent a wallet](https://tempo.xyz/developers/docs/accounts/automations/give-an-agent-a-wallet) guide; a session also needs the payment-channel calls in the key's scopes.

> **Important:** Never hardcode private keys in source files. Always read from an environment variable or secure store.

See [making-requests](making-requests.md) for full usage examples.
