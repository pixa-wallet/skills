---
name: pixa-wallet
description: Pay for x402-gated APIs and manage an Algorand wallet through the Pixa Wallet MCP server. Use when an HTTP request returns 402 Payment Required, when the user asks to send USDC or ALGO, check balances, swap tokens on Tinyman, mint an ASA token, audit agent spending, or discover paid AI services on Algorand.
---

# Pixa Wallet

Pixa is an MCP server (`pixa-wallet-mcp` on npm) that gives an AI agent a real
Algorand wallet: it can hold USDC, settle x402 micropayments autonomously, move
funds on-chain, and swap or mint tokens — all guarded by hard budget limits.

## When to use this skill

- A request to a URL fails with **402 Payment Required** → use `x402_fetch`.
- The user asks to **send USDC/ALGO**, **check balance**, **swap**, or **mint a token**.
- The user wants to find **paid AI services** an agent can call (search/discovery).
- You need to **audit what the agent spent** or **top up** the wallet.

## Setup (one time)

The server is published on npm as **`pixa-wallet-mcp`** (do NOT confuse with
`x402-wallet-mcp`, a different project). Three ways to install:

1. **Claude Desktop / any MCP-JSON client** — add to `mcpServers`:

```json
{
  "mcpServers": {
    "pixa": {
      "command": "npx",
      "args": ["-y", "pixa-wallet-mcp"],
      "env": {
        "ALGORAND_MNEMONIC": "your 25-word mnemonic",
        "NETWORK": "algorand-testnet",
        "MAX_PER_CALL": "0.10",
        "MAX_PER_DAY": "20.00"
      }
    }
  }
}
```

2. **Claude Code** — `claude mcp add pixa -e ALGORAND_MNEMONIC="..." -e NETWORK=algorand-testnet -- npx -y pixa-wallet-mcp`
3. **One-click desktop extension** — download the `.mcpb` from https://pixawallet.xyz and double-click.

### Environment variables

| Variable | Required | Default | Meaning |
|---|---|---|---|
| `ALGORAND_MNEMONIC` | for Algorand | — | 25-word Algorand seed phrase |
| `NETWORK` | no | `algorand-testnet` | `algorand` (mainnet) or `algorand-testnet` |
| `MAX_PER_CALL` | no | `0.10` | Max USD per single payment |
| `MAX_PER_DAY` | no | `20.00` | Daily spending cap (USD) |
| `EVM_PRIVATE_KEY` | optional | — | Enables paying on Base / Base Sepolia |
| `STELLAR_SECRET` | optional | — | Enables paying on Stellar |

With no keys set the server runs in `READ_ONLY` mode: payment tools return
"No wallet configured" errors. With only `ALGORAND_MNEMONIC` set the mode is
`ALGORAND_ONLY` — the common case.

## Golden path: calling a paid API

Prefer **`x402_fetch`** — it does the whole dance in one call:
request → receive 402 challenge → check budget → sign USDC authorization →
retry with `X-PAYMENT` header → return the final response.

```
x402_fetch { url: "https://pixa-api.vercel.app/weather/current?city=Tokyo" }
```

Notes:
- Works for `GET/POST/PUT/DELETE/PATCH`; pass `headers` / `body` (string) as needed.
- Non-402 responses are returned untouched, so it is safe for any URL.
- Binary responses come back base64-encoded (`bodyEncoding: "base64"`).

Use the lower-level **`pay`** tool only when you need the raw `X-PAYMENT`
header value to attach to a request you are constructing yourself:

```
pay { amount: "0.05", recipient: "<payTo address>", network: "algorand-testnet", resource: "<url>" }
```

## Tool reference

### Wallet
| Tool | Arguments | Notes |
|---|---|---|
| `check_balance` | — | Address, mode, USDC + ALGO balances |
| `transfer_usdc` | `to`, `amount`, `network?`, `note?` | Real on-chain ASA transfer (not x402). `to` accepts an address or NFD name (`bob.algo`) |
| `transfer_algo` | `to`, `amount`, `note?`, `network?` | Native ALGO; use to cover fees or seed wallets |
| `spending_report` | — | Spent today/session, remaining budget, payment history |
| `request_funding` | `amount`, `currency?` (USDC\|ALGO), `note?`, `network?` | Returns an ARC-26 `algorand://` deep-link the user can open in Pera Wallet to top up |

### x402 payments
| Tool | Arguments | Notes |
|---|---|---|
| `x402_fetch` | `url`, `method?`, `headers?`, `body?` | Auto-pays 402 challenges |
| `pay` | `amount`, `recipient`, `network`, `resource?` | Returns `X-PAYMENT` header value only |
| `search_bazaar` | `query?` | Discover x402-gated AI services (chat, STT/TTS, image gen, storage, search, weather) |

### DeFi
| Tool | Arguments | Notes |
|---|---|---|
| `tinyman_swap_fixed_input` | `assetInId`, `assetOutId`, `amountIn`, `slippagePct?`, `network?` | Spend exactly X |
| `tinyman_swap_fixed_output` | `assetInId`, `assetOutId`, `amountOut`, `slippagePct?`, `network?` | Receive exactly X — ideal to obtain the exact USDC an x402 call needs |
| `create_token` | `name`, `ticker`, `totalSupply`, `decimals?`, `url?`, `note?`, `freeze?`, `network?` | Mint a new ASA; creator holds full supply |

## Networks and assets

| Network value | Chain | USDC asset |
|---|---|---|
| `algorand` | Algorand mainnet | ASA `31566704` |
| `algorand-testnet` | Algorand testnet | ASA `10458941` |
| `base` / `base-sepolia` | Base (EVM) | needs `EVM_PRIVATE_KEY` |
| `stellar` / `stellar-testnet` | Stellar | needs `STELLAR_SECRET` |

- Asset ID `0` = native ALGO (in swap tools).
- Algorand settles in ~3 seconds with ~0.001 ALGO fees, so micropayments are viable.
- **Tools default to `algorand-testnet`.** Pass `network: "algorand"` explicitly for
  mainnet, and only when the user clearly intends real funds.

## Discovering paid services

- `search_bazaar { query: "image" }` — curated Unified Agent Layer services.
- **Pixa Registry** (trust-scored x402 catalog): `GET https://pixaregistry.vercel.app/api/search?q=<term>`
  returns agent-ready cards (price, network, payTo, trust tier). Prefer services
  with tier `verified`. Sellers register at https://pixawallet.xyz/registry.

## Recipes

- **Paid call but wallet only holds ALGO** → `tinyman_swap_fixed_output` to get the
  exact USDC amount, then `x402_fetch`.
- **Wallet empty** → `request_funding { amount: "5.00" }` and show the returned
  `algorand://` link to the user (Circle faucet funds testnet USDC).
- **"Send $2 to bob.algo"** → `transfer_usdc { to: "bob.algo", amount: "2.00" }` —
  NFD names resolve automatically.
- **Before a batch of paid calls** → `spending_report` to confirm remaining daily budget.

## Budgets and safety

- Every payment passes `MAX_PER_CALL` / `MAX_PER_DAY` checks; exceeding them
  returns an error instead of paying. Do not retry past a budget error — surface
  it to the user and suggest raising the limit or using `request_funding`.
- Never print, log, or echo the mnemonic or private keys. Never ask the user to
  paste a mnemonic into chat; it belongs in the MCP `env` config only.
- USDC is an ASA: the **recipient must be opted in** to the USDC asset or the
  transfer fails. Suggest opting in (0-amount self-transfer) when that happens.
- Keep ~0.1 ALGO in the wallet for transaction fees; swaps and transfers fail
  without gas.

## Troubleshooting

| Symptom | Cause / fix |
|---|---|
| "No wallet configured" | Set `ALGORAND_MNEMONIC` in the server env and restart the client |
| Mode is `READ_ONLY` in `check_balance` | Same — no signing key present |
| Budget error on `pay`/`x402_fetch` | Per-call or daily cap hit — see `spending_report` |
| ASA transfer fails | Recipient not opted into USDC, or sender lacks ALGO for fees |
| Paid on wrong network | Tools default to testnet; pass `network: "algorand"` for mainnet |
