# ViewMax MCP

Official remote MCP server for [ViewMax Studio](https://viewmax.studio/mcp). Generate AI video, images, music, and speech from Claude, Cursor, ChatGPT (Codex), or any MCP client.

**Endpoint:** `https://viewmax.studio/api/mcp`  
**Docs:** https://viewmax.studio/docs/mcp

## Connect

### Claude (Web & Desktop)

Settings → Connectors → Add custom connector. Name it `ViewMax`, paste:

```text
https://viewmax.studio/api/mcp
```

Click Connect and sign in with your ViewMax account. OAuth, no API key.

### Claude Code

```bash
claude mcp add --transport http viewmax https://viewmax.studio/api/mcp --header "Authorization: Bearer YOUR_API_KEY"
```

### Cursor

Add to `~/.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "viewmax": {
      "url": "https://viewmax.studio/api/mcp",
      "headers": {
        "Authorization": "Bearer YOUR_API_KEY"
      }
    }
  }
}
```

### Codex (ChatGPT)

```bash
export VIEWMAX_API_KEY=YOUR_API_KEY
codex mcp add viewmax --url https://viewmax.studio/api/mcp --bearer-token-env-var VIEWMAX_API_KEY
```

Create an API key in ViewMax Settings → API Keys.

## Tools

| Tool | Purpose |
| --- | --- |
| `list_video_models` / `get_video_model` | Discover video models and pricing |
| `generate_video` | Create a video generation task |
| `list_image_models` / `get_image_model` | Discover image models and pricing |
| `generate_image` | Create an image generation task |
| `list_music_models` | Discover music models |
| `generate_music` | Generate music (usually returns URLs immediately) |
| `list_voices` | List TTS voices |
| `generate_speech` | Text to speech |
| `generate_sound_effect` | Sound effect from a prompt |
| `get_task` / `wait_for_task` | Poll generation status |
| `get_credits` | Remaining account credits |

Generation consumes ViewMax credits. Inspect a model before calling a `generate_*` tool.

## License

Documentation in this repository is provided so agents and directories can discover the hosted ViewMax MCP server. The ViewMax product itself is a hosted service at [viewmax.studio](https://viewmax.studio).
