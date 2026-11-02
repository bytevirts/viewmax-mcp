---
name: viewmax
description: Generate AI video, images, music, speech and sound effects with the official ViewMax Studio (viewmax.studio) remote MCP server. Use when the user wants to create or edit videos (text-, image- or video-to-video), keep characters consistent across a multi-shot long video, generate or edit images, compose music, do text-to-speech or sound effects, check generation tasks, or check ViewMax credits.
---

# ViewMax Studio MCP

This is the **official** ViewMax MCP server, run by ViewMax Studio ([viewmax.studio](https://viewmax.studio)) and published by bytevirts as `io.github.bytevirts/viewmax` in the official MCP Registry. The only official endpoint is `https://viewmax.studio/api/mcp`.

## When to use

Use this skill when the user asks to:

- make a video from a prompt, a still image or an existing video, including multi-shot or longer videos with the same characters
- create an image from text, or edit or restyle an image
- compose an original music track (instrumental, or with lyrics)
- turn text into speech, or create a sound effect
- check a running ViewMax task, or how many ViewMax credits are left

## Connect

- **Transport:** Streamable HTTP at `https://viewmax.studio/api/mcp`
- **Auth:** OAuth (claude.ai / Claude Desktop custom connector), or the header `Authorization: Bearer $VIEWMAX_API_KEY`. Create the key in ViewMax Settings → API Keys.
- `initialize`, `tools/list` and the catalog tools work without a token. Generation, task and credit tools need auth.

Claude Desktop / claude.ai: Settings → Connectors → Add custom connector → name `ViewMax`, URL `https://viewmax.studio/api/mcp` → Connect.

Claude Code:

```bash
claude mcp add --transport http viewmax https://viewmax.studio/api/mcp --header "Authorization: Bearer $VIEWMAX_API_KEY"
```

Cursor (`~/.cursor/mcp.json`):

```json
{"mcpServers":{"viewmax":{"url":"https://viewmax.studio/api/mcp","headers":{"Authorization":"Bearer ${env:VIEWMAX_API_KEY}"}}}}
```

Codex:

```bash
codex mcp add viewmax --url https://viewmax.studio/api/mcp --bearer-token-env-var VIEWMAX_API_KEY
```

Full docs: https://viewmax.studio/mcp

## The 14 tools

| Tool | What it does | Cost |
| --- | --- | --- |
| `list_video_models` | Lists video models and their capabilities | read-only |
| `get_video_model(model)` | Gets modes, durations, resolutions, aspect ratios, audio toggle, credit pricing, defaults and `max_prompt_length` | read-only |
| `generate_video(model, prompt, …)` | Creates a video task and returns `task_id`. Optional: `mode`, `image_urls`, `video_urls`, `duration`, `resolution`, `aspect_ratio`, `audio`, `source_video_duration_seconds`, `idempotency_key` | credits |
| `list_image_models` | Lists image models with modes (text-to-image, image-to-image), aspect ratios, qualities and credit labels | read-only |
| `get_image_model(model)` | Gets full capabilities and credit pricing for one image model | read-only |
| `generate_image(model, prompt, …)` | Creates an image task. Optional: `scene`, `image_urls` (image-to-image), `aspect_ratio`, `quality`, `idempotency_key` | credits* |
| `list_music_models` | Lists music models, controls (duration, instrumental) and cost | read-only |
| `generate_music(prompt, …)` | Creates an original track. Optional: `duration_seconds` (3–300, default 60), `instrumental`, `style`, `lyrics`, `idempotency_key` | 60 credits up to 60 s, then 1 credit/s |
| `list_voices` | Lists TTS voices (id, name, gender, accent, language) and speech pricing | read-only |
| `generate_speech(text, voice_id, …)` | Text to speech. Optional: `model_id`, `speed` (0.25–4), `stability`, `similarity_boost`, `idempotency_key` | ceil(chars/20) credits, min 1 |
| `generate_sound_effect(prompt, …)` | Sound effect from a prompt of up to 450 characters. Optional: `duration_seconds` (0.5–22), `prompt_influence`, `idempotency_key` | 5 credits |
| `get_task(task_id)` | Gets status, `output_urls` and `cost_credits` for a task | read-only |
| `wait_for_task(task_id, timeout_seconds?)` | Waits up to 1–50 s (default 45) and returns the latest status | read-only |
| `get_credits` | Gets the account's remaining credits | read-only |

\* Some models cost 0 credits within a daily fair-use pool on certain plans (described in the tool descriptions). The tool's own description and `get_*_model` are always the source of truth for pricing.

## Rules

1. **Check before you spend.** Call `get_video_model` / `get_image_model` (or the `list_*` tools) before any `generate_*` call. Omitted `duration`, `resolution` and `aspect_ratio` use the mode's defaults and are billed at those values. Tell the user the expected cost and check `get_credits` before large jobs.
2. **Always send an `idempotency_key`** (1–200 visible ASCII characters, no spaces) for a generation, and reuse it when you retry after a timeout. A repeated key returns the original task and is not charged again.
3. **Poll, don't regenerate.** Video and image calls return a `task_id`. Keep calling `wait_for_task` until `status` is `success`, `failed` or `canceled`. Some videos take several minutes. Results are persisted, so a slow task is not a lost task. Failed or canceled tasks include `error_code` / `error_message` and are refunded automatically.
4. Music, speech and sound effects render synchronously and usually return `output_urls` right away. If they don't, use `get_task`.
5. Respect `max_prompt_length` when a model reports it.

## Workflows

### Long video with consistent characters

1. `list_video_models` → `get_video_model` to pick a model that supports image-to-video, and note its durations, aspect ratios and per-clip credits.
2. Make a character reference sheet with `generate_image` (describe the face, clothing and colors in detail) → `wait_for_task` → keep the output URL.
3. Write a shot list. For each shot, call `generate_video` with that reference in `image_urls`, the same character description in every prompt, the same `aspect_ratio` and `resolution`, and a unique `idempotency_key` such as `story1-shot03`.
4. Optionally restyle or extend an existing clip with `video_urls` (video-to-video). Pass `source_video_duration_seconds` when the model's pricing depends on it.
5. Run `wait_for_task` on each `task_id`, then collect the `output_urls` in shot order. Add narration with `generate_speech`, a score with `generate_music` and effects with `generate_sound_effect`, then give the user every URL so they can assemble the edit.

### Image

`list_image_models` → `get_image_model` → `generate_image(model, prompt, aspect_ratio, quality)`. Add `image_urls` to edit or restyle an image. Then `wait_for_task` until it returns result URLs.

### Music

`list_music_models` → `generate_music`. To describe a track, put the description in `prompt` and leave out `style`. Setting `style` sends `prompt` as the song's lyrics, so don't send `lyrics` too. Use `instrumental: true` for no vocals.

### Speech and sound effects

`list_voices` → choose a `voice_id` → `generate_speech(text, voice_id)`. For effects, call `generate_sound_effect(prompt, duration_seconds)`.

### Tasks and credits

Use `get_task` for a one-time status check, `wait_for_task` for blocking polls, and `get_credits` for the remaining balance. Report `cost_credits` from the finished task to the user.
