---
description: Generate an AI video safely via Faste Media MCP with mandatory pre-flight cost confirmation
argument-hint: "<video prompt and/or source image URL/path> [optional: duration / resolution]"
---

Generate a video using the `faste-media` MCP server based on the user's request: `$ARGUMENTS`

Follow this exact safe workflow:
1. If `$ARGUMENTS` is empty, ask the user for the video concept, desired duration/resolution, and whether they have a reference/first-frame image.
2. Call `list_models(modality="video")` to check permitted video models for the current API key.
3. Choose the appropriate model (`minimax-h3-t2v` / `wan-3.0-video-prime` for text-to-video, `minimax-h3-i2v` / `gemini-omni-1.2-flash` / `seedance-2.5` for image-to-video, `minimax-h3-ref2v` for multi-reference video, or `video-upscale` for upscaling).
4. Call `get_model_schema(model_id=...)` to inspect exact parameter names (note enum casing such as `720P` vs `720p` and duration ranges).
5. If animating a local workspace image, upload it via `upload_local_media` first (when using local `faste-mcp`).
6. **MANDATORY COST CHECK:** Call `estimate_cost(model=..., input=...)` and `get_account_balance()`. Report the exact estimated Credits and IDR (`1 Credit = Rp 100`) to the user and **ask for explicit confirmation before submitting** whenever estimated cost exceeds 50 Credits (Rp 5.000) or the user has not pre-approved spending.
7. Once confirmed, submit asynchronously using `generate_media(model=..., input=..., wait=false, idempotency_key="<deterministic-key>")` so long video renders never hit HTTP timeouts.
8. Poll `get_job_status(job_id=...)` at reasonable intervals (every 10–15 seconds) until `succeeded` or `failed`. If the job fails, explain that reserved credits are automatically refunded.
9. Return the white-label CDN video URL (`https://obj.faste.dev/...`), actual credits charged, and note the **30-day CDN retention policy**.
