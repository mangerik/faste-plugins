# Faste Media Plugin for Claude

Official **Faste Media BaaS** plugin for **Claude Desktop, Cowork, and Claude Code**, enabling safe AI image and video generation, editing, upscaling, credit inspection, and QRIS balance top-ups through Faste's remote Model Context Protocol (MCP) server (`https://api.faste.dev/mcp`).

## Data Handling & Privacy

When you connect this plugin via OAuth 2.1, Claude authenticates with `https://api.faste.dev` using short-lived, revocable OAuth access tokens (`faste_at_*`) bound to your Faste API key and its existing scope, model allowlist, and spend guardrails. Tool calls send only the generation prompts, model parameters, optional public media URLs, idempotency keys, or confirmed top-up amounts you request. Faste returns structured job metadata, integer credit balances (`1 Credit = Rp 100`), QRIS payment links (only when `billing:write` is granted and you confirm an amount), and generated image or video URLs hosted on `https://obj.faste.dev` (automatically expired after 30 days). Revoking your source API key in Faste immediately disconnects all OAuth sessions.

Read our full [Privacy Policy](https://faste.dev/privacy) for retention, subprocessor, and data deletion details.

## Included Capabilities

- **Remote MCP Server** (`.mcp.json`): Zero-install Streamable HTTP connection to `https://api.faste.dev/mcp` with automatic **OAuth 2.1 + PKCE (`S256`)** sign-in (9 core tools: `list_models`, `get_model_schema`, `estimate_cost`, `get_account_balance`, `create_topup`, `get_topup_status`, `generate_media`, `get_job_status`, `list_jobs`).
- **Agent Skill** (`skills/faste-media-gen/SKILL.md`): Guides Claude through schema-first parameter validation, pre-flight credit estimation, cost confirmation for expensive jobs, deterministic idempotency keys, async video polling, QRIS top-up safety, and 30-day CDN retention reminders.
- **Slash Commands**:
  - `/generate-image` — Generate or edit an AI image safely with schema and cost checks
  - `/generate-video` — Generate or animate an AI video with mandatory pre-flight cost confirmation
  - `/check-balance` — Inspect credits, concurrency limits, and active API key budget guardrails
  - `/topup-credits` — Generate a QRIS top-up link (requires `billing:write`) and verify settlement

---

## Installation & Sign-In

### Claude Desktop & Cowork

1. Install **`faste-media`** from the Claude plugin catalog (or add marketplace repository **`mangerik/faste-plugins`**).
2. Click **Connect**. Your browser opens `https://api.faste.dev/oauth/authorize`.
3. Paste your Faste API key (`sk_live_...`) and click **Authorize & Connect**.

### Claude Code CLI

```bash
claude plugin marketplace add mangerik/faste-plugins
claude plugin install faste-media@faste-plugins
```

Run `/mcp` inside Claude Code to sign in via OAuth 2.1, then run `/check-balance` to verify your connection and credit balance.

---

## Documentation & Optional Local CLI

This plugin uses Faste's hosted Remote MCP server (`https://api.faste.dev/mcp`) and requires no local binary installation. For full API reference, scoped agent API key setup, or optional local workspace upload/download tools (`faste-mcp`), see the official [Faste Agent Harness Documentation](https://faste.dev/docs#agent-harnesses).
