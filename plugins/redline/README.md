# redline

Configurable statusline for Claude Code with progress bars, git info, cost tracking, and PS1-style prompt.

## Requirements

`jq`. Every component reads its data from the statusline JSON with it. Install
it with `apt install jq`, `brew install jq`, or the equivalent on your platform.
When `jq` is absent, the statusline says so instead of rendering blank.

## Install

```
/plugin marketplace add allixsenos/claude-plugins
/plugin install redline@allixsenos
/reload-plugins
/redline:setup
```

The setup skill writes a version-resilient wrapper, and sets your statusLine setting for you.

## Skills

- `/redline:setup` runs the one-time setup after install
- `/redline:configure` changes the layout interactively
- `/redline:changelog` shows recent Claude Code release notes

## Features

- PS1-style `user@host:cwd` prompt
- Git branch + compact status flags (S=staged, M=modified, ?=untracked)
- Model name display
- Progress bars for context window, 5h session limit, and 7d weekly limit
  - Color thresholds: green < 60%, yellow >= 60%, red >= 80%
- Rate-limit bars carry an in-bar `|` marker for your position in the window.
  Fill past the marker means you burn faster than real time
- Short variants with dark grey brackets (e.g. `[5h 42%]`) flag a burning pace
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
| `asu_cmd` | auto | The command that launches asu. Leave it empty to auto-detect: `npx --yes @allixsenos/asu`, then `bunx`, then `pnpm dlx`, then a global `asu` on PATH. Set it to override, for example with a local checkout. |

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
| `ctx_bar` | Context window usage as a `ctx NN% [bar]` 10-step progress bar. When `CLAUDE_CODE_AUTO_COMPACT_WINDOW` sits below the full window of the model, cells past the cap render as `✘`, which shows the unreachable capacity at a glance. |
| `ctx_short` | Context window usage as colored text in brackets |
| `5h_bar` | 5-hour rate limit: `5h NN% [bar\|with\|marker] countdown`. `\|` shows elapsed-time position |
| `5h_short` | 5-hour rate limit as `[5h NN%]` with `↑` inside when burning hot + countdown |
| `7d_bar` | 7-day rate limit: `7d NN% [bar\|with\|marker] countdown` |
| `7d_short` | 7-day rate limit as `[7d NN%]` with `↑` inside when burning hot + countdown |
| `fable_bar` | Fable weekly rate limit, as `fable NN% [bar\|with\|marker] countdown`. Off by default, and needs `asu`. See [Fable weekly limit](#fable-weekly-limit). |
| `fable_short` | Fable weekly rate limit as `[fable NN%]`, with the burn arrow inside and the countdown after |
| `cost` | Session cost in yellow (e.g. `$0.42`) |
| `lines` | Lines added (green) and removed (red) |
| `update` | Shows bold yellow `↑ claude code A.B.C → X.Y.Z` when the latest on npm is newer than the version that runs this session. It reads the running version from the parent process of the statusline, which is the Claude Code binary that started this session, at the path `.../versions/X.Y.Z/claude`. It does not read `claude --version` on PATH. Claude self-updates in the background, so PATH always points at the latest copy on disk. A long-running session keeps its launch version until you restart it, and this component flags exactly that. It stays silent when the session is current, and when the session binary sits outside a versioned path, such as a custom install. It checks npm at most once every 4 hours, and caches the answer in `/tmp/redline-claude-version`. |

Put a component on any line, in any order. Omit a component to turn it off completely. The script does no work for a component that is absent from the config.

### Fable weekly limit

Fable has its own weekly quota, separate from the all-models weekly limit. The statusline JSON does not report it. Claude Code parses a per-model bucket internally, but it sends only `five_hour`, `seven_day` and `spend_limit` to the statusline hook. So `fable_bar` and `fable_short` read the number from [asu](https://github.com/allixsenos/asu) instead.

This is opt-in and off by default. Nothing runs until you put `fable_bar` or `fable_short` in `lines`.

To turn it on:

1. Make sure one of `npx`, `bunx`, or `pnpm` is on your PATH. Node.js ships `npx`, so most machines pass this step already. You do not need to install asu, because the runner fetches it. A global `asu` from `npm install -g @allixsenos/asu` also works, and it is the last fallback.
2. Add `fable_bar` or `fable_short` to a line in your config.

How it behaves:

- asu takes about 0.7 seconds, so the statusline never waits for it. Each render shows the cached value, and starts a background refresh when the cache is older than `fable_ttl`.
- The first render after the cache goes cold shows nothing, and the next render shows the number.
- Auto-detection prefers a package runner over a global install. The runner needs no install step, and it fetches the current asu release on each cold start. A global `asu` stays at whatever version `npm install -g` last put there.
- The component stays silent when nothing on PATH can launch asu. It also stays silent when asu is not signed in, or when asu is broken. It prints no error, because a statusline is the wrong place for one.
- The cache is `/tmp/redline-asu-<uid>.json`. All your sessions share it, and a lock keeps only one of them refreshing.
- The component drops a cached window whose reset time is in the past, because the percentage would describe the previous week.
