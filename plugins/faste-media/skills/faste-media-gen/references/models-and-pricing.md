# Faste Media Model Catalog & Credit Pricing Reference

All prices use pure integer Credits where **`1 Credit = Rp 100`**.
Always call `list_models` and `get_model_schema` at runtime as the authoritative source (pricing or active status may change, and your API key's `allowed_models` policy may restrict visible models).

---

## Image Models (6 Active Hero Models)

| Model ID | Task Type | Key Parameters | Credit Range | IDR Range | Best For |
|---|---|---|---|---|---|
| `minimax-h3-t2i` | `image.text_to_image` | `prompt`, `resolution` (`1K`/`2K`/`4K`), `aspect_ratio` (or `width`+`height`), optional `expand_prompt`, `output_format` (`png`/`jpeg`), `seed` | 3 – 7 | Rp 300 – Rp 700 | Fast, economical text-to-image (default 2K = 3 Credits) |
| `seedream-5.5-t2i` | `image.text_to_image` | `prompt`, `size` (`1K`/`1.5K`/`2K`), `aspect_ratio`, `output_format` (`png`/`jpeg`), `background` (`auto`/`opaque`/`transparent`) | 5 – 8 | Rp 500 – Rp 800 | Commercial studio images, transparent PNG assets |
| `minimax-h3-image-edit` | `image.image_to_image` | `prompt`, `image_urls` (1–16 URIs), `resolution` (`1K`/`2K`/`4K`), `aspect_ratio`, optional `expand_prompt`, `output_format`, `seed` | 3 – 7 | Rp 300 – Rp 700 | Economical multi-image editing & style transfer |
| `seedream-5.5-i2i` | `image.image_to_image` | `prompt`, `image_urls` (1–10 URIs), `size` (`1K`/`1.5K`/`2K`), `background` | 5 – 8 | Rp 500 – Rp 800 | High-fidelity product restyling / compositing |
| `seedream-5.5-layer-edit` | `image.layer_edit` | `image_url`, optional `prompt`, `size` (`1K`/`1.5K`/`2K`/`auto`), `max_images` (2–100) | 25 – 40 | Rp 2.500 – Rp 4.000 | Decomposing an image into editable transparent layers |
| `image-upscale` | `image.upscale` | `image_url`, `upscale_factor` (`1`/`2`/`4`/`8`, default `2`) | 3 – 18 | Rp 300 – Rp 1.800 | Upscaling generated or user-provided images |

### Image Parameter Gotchas
- **`minimax-h3-t2i` & `minimax-h3-image-edit`**: Do **not** combine `width`/`height` with `resolution` or `aspect_ratio` (the schema rejects mutually exclusive combinations). Uppercase resolution enum: `"1K"`, `"2K"`, `"4K"`.
- **`seedream-5.5-*`**: Uses `size` (`"1K"`, `"1.5K"`, `"2K"`) instead of `resolution`, and `image_urls` (array) for `seedream-5.5-i2i` vs `image_url` (single string) for `seedream-5.5-layer-edit`.

---

## Video Models (10 Active Hero Models)

> Always run `estimate_cost` and confirm with the user before submitting video jobs. Submit with `wait=false` and poll `get_job_status`.

| Model ID | Task Type | Key Parameters | Starting Cost | Best For |
|---|---|---|---|---|
| `minimax-h3-t2v` | `video.text_to_video` | `prompt`, `aspect_ratio` (required: `21:9`/`16:9`/`4:3`/`1:1`/`3:4`/`9:16`), `duration` (required: 4–15s), `resolution` (`768P`/`2K`) | 40 Credits (768P 4s) / 96 Credits (2K 6s) | Fast, cost-effective text-to-video with native stereo audio |
| `minimax-h3-i2v` | `video.image_to_video` | `prompt`, `first_frame_url` and/or `last_frame_url`, `duration` (required: 4–15s), `resolution` (`768P`/`2K`) *(no `aspect_ratio`)* | 40 Credits (768P 4s) / 96 Credits (2K 6s) | Animating a still image with smooth motion & stereo audio |
| `minimax-h3-ref2v` | `video.reference_video` | `prompt`, `duration` (required: 4–15s), at least one of `reference_image_urls` (≤ 9) or `reference_video_urls` (≤ 3), optional `reference_audio_urls` (≤ 3), `aspect_ratio`, `resolution` (`768P`/`2K`) | 48 Credits (768P 4s) / 120 Credits (2K 6s) | Consistent character/subject video from multiple references |
| `magi-2-t2v` | `video.text_to_video` | `prompt` (≤ 15k chars), `duration` (4–30s, default 5s), `aspect_ratio` (`21:9`/`16:9`/`4:3`/`1:1`/`3:4`/`9:16`), `resolution` (`1080p`), `generate_audio` (bool), optional `seed` | 88 Credits (4s) / 110 Credits (5s, 22 Cr/s) | Native 1080p text-to-video with synced dialogue, SFX & ambience |
| `magi-2-i2v` | `video.image_to_video` | `first_frame_url` (required), optional `last_frame_url`, optional `prompt`, `duration` (4–30s, default 5s), `resolution` (`1080p`), `generate_audio` (bool) | 88 Credits (4s) / 110 Credits (5s, 22 Cr/s) | 1080p image-to-video with synced sound & auto aspect ratio |
| `magi-2-ref2v` | `video.reference_video` | `prompt`, at least one of `reference_image_urls` (≤ 10) / `reference_video_urls` (≤ 5) / `reference_audio_urls` (≤ 5), `duration` (4–30s, default 5s), `aspect_ratio`, `resolution` (`1080p`), `generate_audio` | 88 Credits (4s) / 110 Credits (5s, 22 Cr/s) | Identity & voice-consistent 1080p video (flat per-second rate, no ref surcharge) |
| `wan-3.0-video-prime` | `video.text_to_video` | `prompt`, `resolution` (`480P`/`720P`/`1080P`), `duration` (2–30s, default 5s), `audio` (bool), `shot_type` (`single`/`multi`), optional `first_frame_url`/`last_frame_url` | 30 Credits (480P 2s) / 150 Credits (720P 5s) | Multi-shot cinematic video with native audio |
| `gemini-omni-1.2-flash` | `video.image_to_video` | `prompt`, `resolution` (`360p`/`720p`/`1080p`/`4K`), `duration` (3–15s, default 8s), `aspect_ratio` (`16:9`/`9:16`), optional `first_frame_url`, `image_urls` | 50 Credits (360p 3s) / 125 Credits (720p 8s) | Multimodal flash video generation |
| `seedance-2.5` | `video.image_to_video` | `prompt`, `resolution` (`480p`/`720p`/`1080p`/`4K`), `duration` (4–30s or `-1`, default 5s), `generate_audio` (bool), optional `first_frame_url`/`reference_*` | 132 Credits (480p 4s) / 370 Credits (720p 5s) | Flagship high-motion cinematic video with synchronized audio |
| `video-upscale` | `video.video_to_video` | `video_url`, `resolution` (`1080P`/`2K`/`4K`) | 25 / 50 / 95 Credits | Upscaling existing videos to 1080P, 2K, or 4K |

### Video Parameter Gotchas
- **Case-sensitive `resolution` enums**:
  - Uppercase `P`/`K`: `wan-3.0-video-prime` (`480P`, `720P`, `1080P`), `minimax-h3-*` (`768P`, `2K`), `video-upscale` (`1080P`, `2K`, `4K`).
  - Lowercase `p`: `magi-2-*` (`1080p`), `gemini-omni-1.2-flash` (`360p`, `720p`, `1080p`, `4K`), `seedance-2.5` (`480p`, `720p`, `1080p`, `4K`).
- **`minimax-h3` video rules**: `duration` is required on all 3 MiniMax H3 video models (`4`–`15`). `minimax-h3-i2v` derives its ratio from the input frame and does **not** accept `aspect_ratio`; `minimax-h3-t2v` requires an explicit `aspect_ratio` (`adaptive` is not supported); `minimax-h3-ref2v` requires at least one image or video reference (audio alone is not allowed). None of the MiniMax H3 video models accept `prompt_optimizer`.
- **`magi-2-ref2v` reference requirement**: Must include at least one non-empty array among `reference_image_urls`, `reference_video_urls`, or `reference_audio_urls`. `magi-2-i2v` follows the first frame's aspect ratio and does not accept `aspect_ratio`.

---

## Media Retention Policy
- **Input uploads** (`upload_local_media` / `POST /v1/media/upload-url`): retained for **24 hours**.
- **Generated outputs** (`https://obj.faste.dev/outputs/...`): retained for **30 days**.
