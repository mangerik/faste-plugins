---
name: faste-media-gen
description: Generate, edit, upscale, and animate AI images and videos safely using the Faste Media BaaS MCP server (`https://api.faste.dev/mcp` or local `faste-mcp` CLI). Use whenever the user asks to create images, edit photos, animate images into video, upscale media, check media generation job status, or check Faste credit balance.
---

# Faste Media Generation Skill

Use this skill to orchestrate AI Image and Video generation through the **Faste Media MCP server** (`faste-media`) safely, accurately, and cost-consciously.

## Core Principles

1. **Economy:** `1 Credit = Rp 100` (integer credits only). Every job reserves credits atomically before processing and automatically refunds them if generation fails.
2. **Zero Guessing:** Always inspect `list_models` and `get_model_schema` before constructing `generate_media.input`. Models enforce strict JSON Schema validation (`additionalProperties: false`) and case-sensitive resolution enums (e.g., `720p` vs `720P`).
3. **Cost Safety:** Always call `estimate_cost` before calling `generate_media`. For video generation or any job costing **> 50 Credits (Rp 5.000)**, report the exact Credit + IDR cost and obtain explicit user approval unless the user already approved the spend in the current turn.
4. **Idempotency:** Always pass a deterministic `idempotency_key` (e.g., `task-<slug>-v1`) to `generate_media` so retries after network hiccups never double-charge credits.
5. **Security & Least Privilege:** Never read, print, log, or upload secrets (`.env`, API keys, SSH keys, credentials). Never attempt admin actions from an agent session. Only call `create_topup` after the user explicitly confirms the top-up nominal (IDR), and remember it requires the `billing:write` scope.

---

## Standard Generation Workflow

Follow these steps in order:

### Step 1: Check Balance & Guardrails (When Starting a Session or Running Multiple Jobs)
Call `get_account_balance` to see:
- `balance_available` (Credits & IDR) and `balance_reserved`
- `active_jobs_count` vs `max_concurrent_jobs`
- Key-level guardrails: `key_allowed_models`, `key_max_credits_per_job`, `key_daily_spend_used_today`, `key_daily_spend_limit_credits`, `key_expires_at`

### Step 2: Discover Permitted Models & Exact Schema
1. Call `list_models(modality="image" | "video", task_type=...)`.
   - The server automatically filters out models blocked by `allowed_models` on the current API key.
2. Call `get_model_schema(model_id="<chosen-model>")` to verify required fields, enum values, min/max bounds, and `dependentRequired` rules.
   - See `references/models-and-pricing.md` for a quick-reference overview of the hero catalog.

### Step 3: Prepare Input Media (For I2I, I2V, Upscale, Reference-to-Video)
- If the user provides an existing public HTTPS image/video URL, pass it directly in `input`.
- If the user references a **local file** inside the workspace AND the local `faste-mcp` CLI is active:
  1. Call `upload_local_media(file_path="relative/path/to/image.png")`.
  2. Pass the returned `media_url` (`https://obj.faste.dev/...`) into `generate_media.input`.
  - Note: Local uploads are sandboxed to `FASTE_WORKSPACE_ROOT`, verified via magic bytes (`jpeg`, `png`, `webp`, `mp4`, `webm`), and expire after **24 hours**.
- If running Remote MCP only (without local CLI tools), ask the user for a public HTTPS URL or instruct them to upload via `POST /v1/media/upload-url`.

### Step 4: Pre-Flight Cost Estimation
Call `estimate_cost(model="<model_id>", input={...})`.
- Confirm `estimated_credits <= balance_available`.
- Confirm `estimated_credits` respects `key_max_credits_per_job` and remaining `key_daily_spend_limit_credits`.
- Ask the user to confirm if `estimated_credits > 50` (Rp 5.000) or if submitting a batch of jobs.

### Step 5: Submit Job (`generate_media`)
- **Images (fast, ~5–40s):** Call `generate_media(model=..., input=..., wait=true, timeout_sec=90, idempotency_key=...)`.
- **Videos (slower, ~30s–5m):** Call `generate_media(model=..., input=..., wait=false, idempotency_key=...)` to avoid client timeout, then proceed to Step 6.

### Step 6: Monitor Async Jobs (`get_job_status`)
If `status` is `queued` or `processing` (or `timed_out_waiting: true`):
- Poll `get_job_status(job_id="<id>")` every **5–10 seconds** for images or **10–15 seconds** for videos.
- Terminal states:
  - `succeeded`: Output URLs are in `output.urls` (white-label CDN `https://obj.faste.dev/...`).
  - `failed`: Check `error.code` and `error.message`; reserved credits are refunded automatically.

### Step 7: Deliver Output & Retention Notice
- Share the white-label CDN URL(s) and actual credits spent (`actual_credits` and IDR value).
- Always remind the user that output media on `https://obj.faste.dev` is retained for **30 days** — download or persist important assets before expiration.
- If the user wants the output saved to their workspace and local `faste-mcp` is active, call `download_media(media_url="https://obj.faste.dev/...", output_path="assets/output.png", overwrite=false)`.

---

## QRIS Top-Up Workflow (When Balance Is Low or User Requests Top-Up)

1. Check `key_scopes` from `get_account_balance`:
   - If `billing:write` is present: ask the user to confirm the desired top-up nominal in Rupiah (multiple of Rp 100, default minimum Rp 25.000, maximum Rp 496.200 per QRIS transaction; `1 Credit = Rp 100`).
   - If `billing:write` is **not** present (e.g., restricted `agent` preset key): explain that this key cannot generate invoices and ask the account owner to top up with a `billing:write`-enabled key.
2. Once the user confirms the nominal, call `create_topup(amount_idr=<nominal>, idempotency_key="topup-<slug>")`.
3. Present the returned **`payment_url`** (clickable QRIS payment link), `credits_granted`, `base_amount_idr`, `gateway_fee_idr`, `total_payable_idr`, `order_id`, and `expires_at` clearly so the user can open and scan QRIS manually.
4. Once the user confirms they have paid, call `get_topup_status(order_id="<order_id>")` (which auto-reconciles with the payment gateway) or `get_account_balance()` to verify `status == "paid"` and updated credits before resuming `generate_media`.

---

## Error Handling Guide

| Error Code | Meaning & Recommended Agent Action |
|---|---|
| `insufficient_credits` | Account balance is too low. Report required vs available credits. If the key has `billing:write`, offer to generate a QRIS payment link via `create_topup` after the user confirms the nominal; otherwise ask the account owner to top up. |
| `insufficient_scope` | Current API key lacks the required scope (e.g., `agent` preset key calling `create_topup` without `billing:write`, or read-only key calling `generate_media`). Inform the user without retrying. |
| `key_policy_violation` | Blocked by key policy (`allowed_models`, `max_credits_per_job`, `daily_spend_limit_credits`, or disallowed Origin/sandbox path). Report the exact policy constraint and suggest a cheaper/allowed model or lower resolution/duration. |
| `invalid_parameters` | Payload failed model JSON Schema validation. Re-read `get_model_schema` and fix parameter names/types/enums before retrying. |
| `concurrency_limit_exceeded` | Too many active jobs running concurrently. Wait for existing jobs (`list_jobs(status="processing")`) to finish before submitting more. |
| `idempotency_conflict` | Same `idempotency_key` reused with a different payload. Generate a new `idempotency_key` for the modified request. |
| `content_policy_violation` | Prompt or input media was rejected by safety filters. Inform the user and adjust the prompt safely. |
| `generation_failed` / `upstream_timeout` | Generation failed and reserved credits were automatically refunded. Safe to retry once with a new `idempotency_key` if the user agrees. |

---

## References
- Model catalog & pricing guide: `references/models-and-pricing.md`
- Agent security & API key policy guide: `references/security-and-policies.md`
