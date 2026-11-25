---
description: Inspect Faste Media account credit balance, tier concurrency limits, and active API key budget guardrails
---

Call `get_account_balance()` on the `faste-media` MCP server and summarize the result clearly for the user:

1. **Account Balance:** Available Credits (and IDR equivalent at `1 Credit = Rp 100`) and Reserved Credits currently held by in-flight jobs.
2. **Account Tier & Concurrency:** Tier name, account status, RPM limit, and `active_jobs_count / max_concurrent_jobs`.
3. **API Key Guardrails (if set):**
   - Permitted scopes (`key_scopes`)
   - Model allowlist (`key_allowed_models`, or "all active models" if unset)
   - Per-job credit cap (`key_max_credits_per_job`)
   - Daily spend usage vs daily cap (`key_daily_spend_used_today / key_daily_spend_limit_credits`)
   - Key expiration timestamp (`key_expires_at`), warning if expiring soon.

Never print or echo the raw `FASTE_API_KEY` value. If balance is low:
- If `billing:write` is included in `key_scopes`, offer to generate a QRIS payment link via `create_topup` after the user confirms the Rupiah nominal (minimum Rp 25.000, maximum Rp 496.200, multiple of Rp 100), then verify settlement via `get_topup_status`.
- If `billing:write` is **not** in `key_scopes` (e.g., restricted `agent` preset key), inform the user that top-ups must be initiated using a key with `billing:write` permission.
