# git-governor

A Claude Code plugin that enforces git governance through PreToolUse hooks. Install it once, and stop worrying that Claude will destroy your git history.

## Rules

| Rule | Default | Description |
|------|---------|-------------|
| `no-amend` | deny | Block `git commit --amend` |
| `no-commit-on-protected` | deny | Block `git commit` on protected branches |
| `no-push-to-protected` | **ask** | Prompt before `git push` targeting protected branches |
| `no-force-push` | deny | Block `--force`, `--force-with-lease`, `+refs` push syntax |
| `no-reset-hard` | deny | Block `git reset --hard` |
| `no-discard-all` | deny | Block `git checkout .`, `git restore .`, `git clean -f` |
| `no-rebase-on-protected` | deny | Block `git rebase` while on a protected branch |
| `no-add-all` | deny | Block `git add .` and `git add -A` (require explicit file paths) |
| `no-merge-pr` | **ask** | Require approval before `gh pr merge` (prevents accidental merges) |
| `require-git-repo` | **allow** | Block `Write`/`Edit` to files outside a git repo (opt-in) |

Protected branches default to `main` and `master`.

The `require-git-repo` rule blocks file edits to any path outside a git repository. A project opts out with a phrase such as "will not use git" in its `CLAUDE.md`.

## Install

```
/plugin install allixsenos/claude-git-governor
```

## Configuration

Configure interactively with the built-in skill:

```
/git-governor:git-governor                              # show effective config
/git-governor:git-governor set no-amend ask             # set a rule
/git-governor:git-governor set no-add-all allow --global  # set globally
/git-governor:git-governor protect release/*            # add protected branch
/git-governor:git-governor reset                        # remove project config
```

Or drop a `.claude/git-governor.json` in your project root to override defaults:

```json
{
  "protected-branches": ["main", "master", "release/*"],
  "rules": {
    "no-amend": "deny",
    "no-force-push": "deny",
    "no-commit-on-protected": "ask",
    "no-push-to-protected": "ask",
    "no-reset-hard": "deny",
    "no-discard-all": "ask",
    "no-rebase-on-protected": "allow",
    "no-add-all": "ask",
    "no-merge-pr": "ask",
    "require-git-repo": "deny"
  }
}
```

Glob patterns work for branch names (`release/*` matches `release/1.0`).

### Rule modes

Every rule supports three modes:

| Mode | Effect |
|------|--------|
| `"deny"` | Hard block. The tool call does not run |
| `"ask"` | Prompt the user to confirm first |
| `"allow"` | Off. No check runs |

Use `"deny"` for an operation that must never happen, such as a force push or `reset --hard`. Use `"ask"` for an operation that needs a human checkpoint, such as a commit on a protected branch, or a change you discard. The hook treats an invalid value as an error, and blocks the call.

### Config precedence

| Scope | Path | Precedence |
|-------|------|------------|
| Project | `<project>/.claude/git-governor.json` | Highest |
| Global | `~/.claude/git-governor.json` | Middle |
| Defaults | Built into the hook | Lowest |

The hook resolves each rule on its own: project, then global, then default. For `protected-branches`, the first config that sets the key wins outright. Nothing merges.

## License

MIT
