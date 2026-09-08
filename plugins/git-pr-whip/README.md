# git-pr-whip

Nudges Claude about PR hygiene at commit time and at push time. It also blocks the one workflow where the nudge would arrive too late.

## What it catches

Three nudges and one block. Each nudge arrives as `additionalContext`, so Claude sees it and you do not.

- **After `git commit`.** *"Before you push, run `gh pr view` and check that the PR is still open."* This catches a PR that someone squash-merged while Claude worked, which leaves the branch stale.
- **After `git push`.** *"Did these commits expand the PR's scope? Update the title and description with `gh pr edit`."* This catches *"while I'm here"* drift. A reviewer opens a PR titled *"fix null session"* and finds a logger refactor.
- **After `git push`.** *"If you addressed review feedback, post a recap comment with what you accepted vs. dismissed."* This gives reviewers a record, so they do not have to rebuild it from the diff.
- **Before `git commit && git push`.** Denied. The commit-time reminder lands only *between* tool calls. A push chained behind the commit gets the reminder too late. Claude must split the chain into two calls.

Detection ignores `git commit` and `git push` inside a quoted string or a heredoc body, and it leaves `git stash push` alone.

## Install

```
/plugin marketplace add allixsenos/claude-plugins
/plugin install git-pr-whip@allixsenos
/reload-plugins
```

No configuration.

## Why a hook and not CLAUDE.md

Because Claude forgets. A compaction eats memory and CLAUDE.md, or whatever sits on screen outweighs them. A hook fires while the reminder can still change the outcome. That means right after the commit, and right before the next push. It is much harder to skip.

## It caught its own author five minutes in

While I wrote the original README for this plugin, the user squash-merged its own landing PR ([#19](https://github.com/allixsenos/claude-plugins/pull/19)). My next commit pushed to a branch that was already dead. The reminder I wrote minutes earlier caught it. Here is the verbatim session:

```
❯ add a faux reproduction of the scenario that the plugin prevents to the plugin's README

  Committed eed6a5, pushed to git-pr-whip, ran 1 bash command

Plugin fired on its own commit — nice end-to-end proof. Let me honor reminder #1
retroactively and confirm PR #19 is still open (not closed/merged while I was
committing).

  Ran 1 bash command

Oh — PR #19 is already MERGED. You merged it while I was working on the README
update, so I just pushed a new commit to a dead branch. This is literally the
scenario reminder #1 exists to prevent. Let me verify the damage and apply the fix.

  Ran 1 bash command

Confirmed: PR #19 was squash-merged as 7924c49 on master. My eed6a58 is orphaned
on the dead branch. Applying the fix the plugin prescribes.

  Cherry-picked 3b0095, created PR #20, ran 1 bash command

Follow-up PR opened: #20.

What just happened:
1. You squash-merged PR #19 → master now has commit 7924c49.
2. My README update committed and pushed to the old git-pr-whip branch — which
   was now dead. eed6a58 was orphaned (not in the PR, not on master).
3. The plugin's own PostToolUse reminder fired on that commit, telling me to run
   gh pr view — which showed state: MERGED.
4. I pulled master, cut a fresh branch git-pr-whip-readme-repro off 7924c49,
   cherry-picked eed6a58 onto it, pushed, opened #20.

The plugin caught its own author on its first real commit. Hard to ask for a
better smoke test.

✻ Churned for 2m 18s
```

Without the nudge, the orphaned commit would sit on the dead branch until someone noticed its absence from `master`. With the nudge, recovery cost three extra commands and no lost work.

## Pairs well with

- **git-governor** handles the dangerous operations. That means amends on pushed commits, force pushes, and commits to `main`. git-pr-whip handles the workflow hygiene that governor does not block.
