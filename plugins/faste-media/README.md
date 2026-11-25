# Faste Media Plugin for Claude

Official **Faste Media BaaS** plugin for **Claude Desktop, Cowork, and Claude Code**, enabling safe AI image and video generation, editing, upscaling, credit inspection, and QRIS balance top-ups through Faste's remote Model Context Protocol (MCP) server (`https://api.faste.dev/mcp`).

## Data Handling & Privacy

When you connect this plugin via OAuth 2.1, Claude authenticates with `https://api.faste.dev` using short-lived, revocable OAuth access tokens (`faste_at_*`) bound to your Faste API key and its existing scope, model allowlist, and spend guardrails. Tool calls send only the generation prompts, model parameters, optional public media URLs, idempotency keys, or confirmed top-up amounts you request. Faste returns structured job metadata, integer credit balances (`1 Credit = Rp 100`), QRIS payment links (only when `billing:write` is granted and you confirm an amount), and generated image or video URLs hosted on `https://obj.faste.dev` (automatically expired after 30 days). Revoking your source API key in Faste immediately disconnects all OAuth sessions.

## Included Capabilities

- **Remote MCP Server** (`.mcp.json`): Zero-install Streamable HTTP connection to `https://api.faste.dev/mcp` with automatic **OAuth 2.1 + PKCE (`S256`)** sign-in (9 core tools: `list_models`, `get_model_schema`, `estimate_cost`, `get_account_balance`, `create_topup`, `get_topup_status`, `generate_media`, `get_job_status`, `list_jobs`).
- **Agent Skill** (`skills/faste-media-gen/SKILL.md`): Guides Claude through schema-first parameter validation, pre-flight credit estimation, cost confirmation for expensive jobs, deterministic idempotency keys, async video polling, QRIS top-up safety, and 30-day CDN retention reminders.
- **Slash Commands**:
  - `/generate-image` — Generate or edit an AI image safely with schema & cost checks
  - `/generate-video` — Generate or animate an AI video with mandatory pre-flight cost confirmation
  - `/check-balance` — Inspect credits, concurrency limits, and active API key budget guardrails
  - `/topup-credits` — Generate a QRIS top-up link (requires `billing:write`) and verify settlement

---

## Installation

### Option 1: Claude Desktop / Cowork (Public Marketplace — Recommended)

1. Open **Plugins → Add marketplace → Add from a repository**.
2. Enter **`mangerik/faste-plugins`** and add the marketplace.
3. Install **`faste-media`** and click **Connect**.
4. Your browser opens `https://api.faste.dev/oauth/authorize` — paste your Faste API key (`sk_live_...`) and click **Authorize & Connect**.

### Option 2: Claude Code CLI (Marketplace)

```bash
claude plugin marketplace add mangerik/faste-plugins
claude plugin install faste-media@faste-plugins
```

Then run `/mcp` inside Claude Code to sign in with OAuth 2.1, or verify with `/check-balance`.

### Option 3: Zero-GitHub 1-Line Installer (`faste.dev`)

```bash
curl -fsSL https://faste.dev/install-skill.sh | sh -s -- --harness claude --plugin
claude --plugin-dir ~/.faste/plugins/claude-code
```

---

## Optional: Local Workspace File Upload & Download (`faste-mcp`)

The default remote MCP server requires zero local binaries. If you also want Claude Code to **upload local workspace images** (`upload_local_media`) or **download generated assets directly to disk** (`download_media`), install the prebuilt `faste-mcp` stdio CLI:

```bash
curl -fsSL https://faste.dev/install-mcp.sh | sh
```

Then configure a project `.mcp.json` using stdio transport:

```json
{
  "mcpServers": {
    "faste-media": {
      "type": "stdio",
      "command": "/Users/you/.local/bin/faste-mcp",
      "env": {
        "FASTE_API_KEY": "${FASTE_API_KEY}",
        "FASTE_API_BASE_URL": "https://api.faste.dev",
        "FASTE_WORKSPACE_ROOT": "${PWD}",
        "FASTE_ALLOWED_CDN_HOSTS": "obj.faste.dev"
      }
    }
  }
}
```

Full documentation & security guidelines: https://faste.dev/docs#agent-harnesses
