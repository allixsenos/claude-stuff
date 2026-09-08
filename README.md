# allixsenos Claude Code plugins & skills

A collection of plugins and standalone skills for Claude Code.

## Setup

**Plugins** (hooks and skills, so they need the plugin system):
```
/plugin marketplace add allixsenos/claude-plugins
```

**Standalone skills** (no hooks, and they work everywhere):
```
npx skills add allixsenos/claude-plugins
```

Use `--list` to see available skills, `--skill <name>` to install a specific one.

## Plugins

Plugins include hooks that run on their own, so they need `/plugin install`.

| Plugin | Description | Install |
|--------|-------------|---------|
| [git-governor](plugins/git-governor/) | Git governance through PreToolUse hooks. Blocks or prompts on amends, force pushes, and protected branches | `/plugin install git-governor@allixsenos` |
| [redline](plugins/redline/) | Configurable statusline with progress bars, git info, and cost tracking | `/plugin install redline@allixsenos` |
| [clockwork](plugins/clockwork/) | Periodic time injection, so Claude knows the day and the time | `/plugin install clockwork@allixsenos` |
| [git-pr-whip](plugins/git-pr-whip/) | PostToolUse reminders after `git commit`. Checks whether the PR closed, and posts review-feedback recaps | `/plugin install git-pr-whip@allixsenos` |

## Skills

Standalone skills, pure knowledge and no hooks. Install one with `npx skills add`, or copy its SKILL.md into the `.claude/skills/` directory of your project.

| Skill | Description | Install |
|-------|-------------|---------|
| [nano-banana](skills/nano-banana/) | Generates and edits images through the Gemini API (Nano Banana models) | `npx skills add allixsenos/claude-plugins --skill nano-banana` |
| [openrouter-imagegen](skills/openrouter-imagegen/) | Generates images through the unified API of OpenRouter, with runtime model discovery | `npx skills add allixsenos/claude-plugins --skill openrouter-imagegen` |
| [linkedin-data-portability](skills/linkedin-data-portability/) | Exports LinkedIn member data (connections, profile, messages, and more) through the DMA Data Portability API | `npx skills add allixsenos/claude-plugins --skill linkedin-data-portability` |
