---
description: Generate an AI image safely via Faste Media MCP with schema validation and cost check
argument-hint: "<prompt or brief description> [optional: aspect ratio / model preference]"
---

Generate an image using the `faste-media` MCP server based on the user's request: `$ARGUMENTS`

Follow this exact safe workflow:
1. If `$ARGUMENTS` is empty, ask the user what image they want to generate (subject, style, aspect ratio).
2. Call `list_models(modality="image")` to see active image models permitted by the current API key.
3. Select the best matching model (e.g., `minimax-h3-t2i` for fast low-cost 2K images or `seedream-5.5-t2i` for rich studio visuals / transparent backgrounds; use `seedream-5.5-i2i` or `minimax-h3-image-edit` if editing an input image).
4. Call `get_model_schema(model_id=...)` to verify exact required and optional fields before constructing `input`.
5. If the user references a local workspace image file, upload it first using `upload_local_media` (when using local `faste-mcp`) or ask for an HTTPS URL. Never read or upload dotfiles or secrets.
6. Call `estimate_cost(model=..., input=...)` and verify the account has sufficient balance via `get_account_balance` if unsure.
7. Submit with `generate_media(model=..., input=..., wait=true, timeout_sec=90, idempotency_key="<deterministic-key>")`.
8. If `timed_out_waiting` is true or status is still `queued`/`processing`, poll `get_job_status(job_id=...)` every 5–10 seconds until `succeeded` or `failed`.
9. Present the final white-label CDN URL(s) (`https://obj.faste.dev/...`), credits used (`1 Credit = Rp 100`), and remind the user that generated output URLs are retained for **30 days** (offer `download_media` if local MCP is active).
