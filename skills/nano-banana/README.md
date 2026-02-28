# Nano Banana

Generate and edit images using the Gemini API directly from Claude Code. No SDKs, no dependencies beyond `curl` and `python3`.

## What it does

- **Generate images** from text prompts via Gemini's image generation models
- **Edit existing images** (remove backgrounds, change colors, add elements)
- **Generate multiple variations** in parallel
- **Verify dimensions** and preview results

Supports 14 aspect ratios (1:1, 16:9, 9:16, etc.), multiple resolutions (1K/2K/4K), and three model tiers at different price points.

## Models

| Model | Name | Best for | Cost |
|-------|------|----------|------|
| `gemini-2.5-flash-image` | Nano Banana | Fast drafts | ~$0.04/image |
| `gemini-3-pro-image-preview` | Nano Banana Pro | High quality, text rendering | ~$0.10/image |
| `gemini-3.1-flash-image-preview` | Nano Banana 2 | Latest flash, 512px support | ~$0.04/image |

## Installation

1. Copy the skill into your Claude Code skills directory:

```bash
cp -r skills/nano-banana ~/.claude/skills/nano-banana
```

2. Get a Gemini API key at https://aistudio.google.com/apikey

3. Set the key in your environment. Pick one:

**Option A — Claude Code settings** (recommended, keeps it out of your shell):
```json
// ~/.claude/settings.json
{
  "env": {
    "GEMINI_API_KEY": "your-key-here"
  }
}
```

**Option B — Shell environment:**
```bash
# ~/.zshrc, ~/.bashrc, or ~/.config/fish/config.fish
export GEMINI_API_KEY="your-key-here"
```

4. Restart Claude Code. Ask it to generate an image — the skill activates automatically.

## Usage

Just ask Claude Code to generate images:

- "generate a hero image for my blog post about Kubernetes"
- "create a 1:1 icon of a lightning bolt, flat illustration style"
- "edit this screenshot to remove the background"
- "make 3 variations of a YouTube thumbnail"

## Requirements

- Claude Code CLI
- `curl` (pre-installed on macOS/Linux)
- `python3` (pre-installed on macOS/Linux)
- A Gemini API key (free tier available)

## Notes

Gemini preview model IDs change periodically. If a model stops working, check https://ai.google.dev/gemini-api/docs/models for current availability and update the model IDs in `SKILL.md`.

## License

MIT
