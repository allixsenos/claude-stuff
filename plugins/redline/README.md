# redline

Configurable statusline for Claude Code with progress bars, git info, cost tracking, and PS1-style prompt.

## Requirements

`jq` — every component reads its data from the statusline JSON with it. Install
it with `apt install jq`, `brew install jq`, or your platform equivalent. If it
is missing the statusline says so instead of rendering blank.

## Install

```
/plugin marketplace add allixsenos/claude-plugins
/plugin install redline@allixsenos
/reload-plugins
/redline:setup
```

The setup skill creates a version-resilient wrapper and configures your statusLine setting automatically.

## Skills

- `/redline:setup` — one-time setup after install
- `/redline:configure` — interactively change the layout
- `/redline:changelog` — show recent Claude Code release notes

## Features

- PS1-style `user@host:cwd` prompt
- Git branch + compact status flags (S=staged, M=modified, ?=untracked)
- Model name display
- Progress bars for context window, 5h session limit, and 7d weekly limit
  - Color thresholds: green < 60%, yellow >= 60%, red >= 80%
- Rate-limit bars include an in-bar `|` marker showing how far through the
  window you are — usage fill past the marker = burning faster than real time
- Short variants with dark grey brackets (e.g. `[5h 42%]`) flag burning pace
  with a bright red `↑` inside the brackets
- Rate limit reset countdown (e.g. `4h`, `5d`) rounded up to the coarsest
  nonzero unit
- Session cost and lines changed (+/-)
- Fully configurable layout via JSON
- Any number of output lines

## Configuration

Create `~/.claude/statusline-config.json` or use `/redline:configure`:

```json
{
  "show_reset_at": 0,
  "burn_threshold": 10,
  "lines": [
    ["ps1", "git", "update"],
    ["model", "ctx_short", "5h_short", "7d_short", "cost", "lines"]
  ]
}
```

Override config path with the `CLAUDE_STATUSLINE_CONFIG` env var.

### Settings

| Setting | Default | Description |
|---------|---------|-------------|
| `show_reset_at` | `0` | Show rate limit reset countdown when usage >= this %. `0` = always, `100` = never. |
| `burn_threshold` | `10` | Percentage-point gap above elapsed time that triggers the `↑` burn icon in `5h_short`/`7d_short`. Only fires once the window is at least 20% elapsed. |
| `fable_ttl` | `300` | Seconds between background refreshes of the Fable window. Only `fable_bar` and `fable_short` use it. |
| `asu_cmd` | `asu` | The command that reads the Fable window. Set it to `npx --yes @allixsenos/asu` to run without a global install, or to a path for a local checkout. |

### Available components

| Component | Description |
|-----------|-------------|
| `ps1` | Bold green user@host, colon, bold blue working directory |
| `user` | Bold green username |
| `host_short` | Bold green short hostname |
| `host_long` | Bold green FQDN hostname |
| `cwd` | Bold blue working directory |
| `git` | Yellow branch name + red status flags |
| `model` | Cyan model display name |
| `ctx_bar` | Context window usage as `ctx NN% [bar]` 10-step progress bar. When `CLAUDE_CODE_AUTO_COMPACT_WINDOW` is set below the model's full window, cells past the cap render as `✘` so unreachable capacity is visible at a glance. |
| `ctx_short` | Context window usage as colored text in brackets |
| `5h_bar` | 5-hour rate limit: `5h NN% [bar\|with\|marker] countdown`. `\|` shows elapsed-time position |
| `5h_short` | 5-hour rate limit as `[5h NN%]` with `↑` inside when burning hot + countdown |
| `7d_bar` | 7-day rate limit: `7d NN% [bar\|with\|marker] countdown` |
| `7d_short` | 7-day rate limit as `[7d NN%]` with `↑` inside when burning hot + countdown |
| `fable_bar` | Fable weekly rate limit, as `fable NN% [bar\|with\|marker] countdown`. Off by default, and needs `asu` — see [Fable weekly limit](#fable-weekly-limit). |
| `fable_short` | Fable weekly rate limit as `[fable NN%]`, with the burn arrow inside and the countdown after |
| `cost` | Session cost in yellow (e.g. `$0.42`) |
| `lines` | Lines added (green) and removed (red) |
| `update` | Shows bold yellow `↑ claude code A.B.C → X.Y.Z` when the latest on npm is newer than the version running this session. Reads the running version from the statusline's parent process (the Claude Code binary that launched this session — path format `.../versions/X.Y.Z/claude`), not from `claude --version` on PATH. Claude self-updates in the background, so PATH always points at the latest on disk; a long-running session stays on its launch version until you restart it, and this component flags that specifically. Silent when current or when the session binary isn't at a versioned path (e.g. custom installs). npm checked at most once every 4 hours (cached in `/tmp/redline-claude-version`). |

Components can be placed on any line in any order. Omit a component to disable it entirely — no work is done for components not in the config.

### Fable weekly limit

Fable has its own weekly quota, separate from the all-models weekly limit. The statusline JSON does not report it. Claude Code parses a per-model bucket internally, but it sends only `five_hour`, `seven_day` and `spend_limit` to the statusline hook. So `fable_bar` and `fable_short` read the number from [asu](https://github.com/allixsenos/asu) instead.

This is opt-in and off by default. Nothing runs until you put `fable_bar` or `fable_short` in `lines`.

To turn it on:

1. Install asu with `npm install -g @allixsenos/asu`. Set `asu_cmd` if you prefer `npx` or a local checkout.
2. Add `fable_bar` or `fable_short` to a line in your config.

How it behaves:

- asu takes about 0.7 seconds, so the statusline never waits for it. Each render shows the cached value, and starts a background refresh when the cache is older than `fable_ttl`.
- The first render after the cache goes cold shows nothing, and the next render shows the number.
- The component stays silent when asu is absent, not signed in, or broken. It prints no error, because a statusline is the wrong place for one.
- The cache is `/tmp/redline-asu-<uid>.json`. All your sessions share it, and a lock keeps only one of them refreshing.
- The component drops a cached window whose reset time is in the past, because the percentage would describe the previous week.
