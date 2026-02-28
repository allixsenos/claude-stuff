# claude-stuff

A collection of [Agent Skills](https://docs.anthropic.com/en/docs/claude-code/skills) for Claude Code.

## Skills

| Skill | Description |
|-------|-------------|
| [nano-banana](skills/nano-banana/) | Generate and edit images using Gemini API (Nano Banana models) directly from Claude Code |

## Installation

Each skill has its own README with installation instructions. The general pattern:

```bash
# Copy the skill folder into your Claude Code skills directory
cp -r skills/<skill-name> ~/.claude/skills/<skill-name>
```

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI
- Each skill may have additional requirements (API keys, tools) — check the skill's README

## License

MIT
