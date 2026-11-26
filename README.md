# Faste Plugins Marketplace for Claude

Official public plugin marketplace for **[Faste Media BaaS](https://faste.dev)** (`https://api.faste.dev/mcp`), compatible with **Claude Desktop**, **Cowork**, and **Claude Code**.

Faste provides a single white-label gateway for 13 production AI image and video models with integer Rupiah credits (`1 Credit = Rp 100`), QRIS balance top-ups, interactive **MCP Apps** media preview/playback/download widgets, automatic visual image inspection for Claude, scoped API key guardrails, and native **OAuth 2.1 + PKCE (`S256`)** sign-in.

---

## Install in Claude Desktop / Cowork

1. Open **Plugins → Add marketplace → Add from a repository**.
2. Enter:
   ```text
   mangerik/faste-plugins
   ```
3. Install **`faste-media`** and click **Connect**.
4. Your browser opens `https://api.faste.dev/oauth/authorize` — paste your Faste API key (`sk_live_...`) and click **Authorize & Connect**.

## Install in Claude Code CLI

```bash
claude plugin marketplace add mangerik/faste-plugins
claude plugin install faste-media@faste-plugins
```

Then run `/mcp` to authenticate via OAuth 2.1, or verify with `/check-balance`.

---

## Included Plugins

| Plugin | Version | Description |
| :--- | :---: | :--- |
| **[`faste-media`](./plugins/faste-media)** | `1.2.0` | Remote Streamable HTTP MCP (`https://api.faste.dev/mcp` via OAuth 2.1) with interactive MCP Apps Media Viewer & QRIS Top-Up Card, `faste-media-gen` Agent Skill, and 4 slash commands (`/generate-image`, `/generate-video`, `/check-balance`, `/topup-credits`). |

---

## Security, Privacy & Support

- **OAuth 2.1 + Live Policy Inheritance:** OAuth access tokens (`faste_at_*`) inherit your source API key's exact `scopes`, `allowed_models`, `max_credits_per_job`, `daily_spend_limit_credits`, and `expires_at`. Revoking the API key immediately disconnects all active sessions.
- **Media Retention:** Generated outputs on `https://obj.faste.dev` are automatically deleted after 30 days; uploaded inputs expire after 24 hours.
- **Privacy Policy:** https://faste.dev/privacy
- **Terms of Service:** https://faste.dev/terms
- **Changelog:** https://faste.dev/changelog
- **Documentation:** https://faste.dev/docs#agent-harnesses
