<div align="center">

# Pixa Agent Skills

**Teach any AI agent to pay.**

Agent Skills for the [Pixa Wallet](https://pixawallet.xyz) MCP server — autonomous x402 micropayments,
USDC/ALGO transfers, Tinyman swaps, and token minting on Algorand.

[![Skills](https://img.shields.io/badge/skills.sh-compatible-black?style=flat-square)](https://skills.sh)
[![MCP](https://img.shields.io/badge/MCP-Native-black?style=flat-square)](https://modelcontextprotocol.io)
[![Algorand](https://img.shields.io/badge/Algorand-Mainnet%20%2B%20Testnet-blue?style=flat-square)](https://algorand.co)
[![x402](https://img.shields.io/badge/x402-Supported-green?style=flat-square)](https://www.x402.org)
[![npm](https://img.shields.io/badge/npm-pixa--wallet--mcp-red?style=flat-square)](https://www.npmjs.com/package/pixa-wallet-mcp)
[![License](https://img.shields.io/badge/License-MIT-gray?style=flat-square)](LICENSE)

</div>

---

AI agents can reason, plan, and execute — Pixa lets them **pay**. This repo packages
that capability as an [Agent Skill](https://skills.sh): a `SKILL.md` playbook that
teaches Claude Code, Cursor, Codex, and other agents exactly how to wire up and
drive the Pixa Wallet MCP server.

## Install

```bash
npx skills add ogsamrat/pixa-skills
```

Target a specific agent, skip prompts:

```bash
npx skills add ogsamrat/pixa-skills -a claude-code -y
```

Or copy by hand — the skill is plain markdown:

```bash
cp -r skills/pixa-wallet ~/.claude/skills/
```

> The skill documents the MCP server itself. To actually move money, the agent
> also needs the server installed: `npx -y pixa-wallet-mcp` (see the skill's
> Setup section for Claude Desktop / Claude Code config).

## Skills

| Skill | What it teaches |
|---|---|
| [`pixa-wallet`](skills/pixa-wallet/SKILL.md) | Paying x402 `402 Payment Required` challenges with `x402_fetch`, on-chain USDC/ALGO transfers with NFD name resolution, Tinyman fixed-input/fixed-output swaps, ASA token minting, budget guardrails, funding deep-links, and paid-service discovery via the Pixa Registry |

### What the agent learns

- **One-call paid HTTP** — `x402_fetch` handles 402 → budget check → sign → retry automatically
- **11 wallet tools** with exact argument shapes, defaults, and failure modes
- **Network map** — Algorand mainnet/testnet USDC ASA IDs, Base and Stellar support
- **Safety rails** — per-call/daily budgets, testnet-first defaults, mnemonic hygiene, ASA opt-in gotchas
- **Recipes** — swap-to-exact-USDC before a paid call, Pera Wallet top-up links, spending audits

## Repository layout

```
pixa-skills/
├── README.md
├── LICENSE
└── skills/
    └── pixa-wallet/
        └── SKILL.md      ← the skill (frontmatter + agent playbook)
```

## The Pixa ecosystem

| Resource | Link |
|---|---|
| Website + one-click `.mcpb` install | [pixawallet.xyz](https://pixawallet.xyz) |
| API documentation | [celoref.mintlify.app/api-reference](https://celoref.mintlify.app/api-reference/introduction) |
| Wallet MCP server (source) | [github.com/pixa-wallet/Pixa](https://github.com/pixa-wallet/Pixa) |
| npm package | [pixa-wallet-mcp](https://www.npmjs.com/package/pixa-wallet-mcp) |
| x402 service registry (trust-scored) | [pixaregistry.vercel.app](https://pixaregistry.vercel.app) |
| Register your paid API | [pixawallet.xyz/registry](https://pixawallet.xyz/registry) |

## Why Algorand

| Capability | Algorand | Typical EVM |
|---|---|---|
| Finality | ~3 s | 12 s+ probabilistic |
| Fees | < $0.001 | $0.50–$5.00+ |
| Native USDC | Yes | Often bridged |
| Micropayments | Viable | Gas exceeds value |

## License

[MIT](LICENSE)
