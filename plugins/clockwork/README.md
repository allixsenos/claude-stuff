# clockwork

Keeps Claude oriented in time during long sessions.

Claude has no internal clock. In conversations that span hours, it loses track of time completely. It guesses 3 AM when the real time is noon, or it does not know the day after a context compaction.

![The problem](the-problem.jpg)

Clockwork injects the current day, date, and time into the conversation context. The hook fires on every message you send. It injects the time only after 10 minutes pass since the last injection, so it stays out of the way during a rapid back and forth.

## Install

```
/plugin marketplace add allixsenos/claude-plugins
/plugin install clockwork@allixsenos
/reload-plugins
```

That is all. It needs no configuration.

## How it works

A `UserPromptSubmit` hook runs a shell script:
1. The script reads a timestamp file (`/tmp/claude-clockwork.stamp`).
2. After 10 minutes or more, it injects `Current time: Tuesday, 2026-04-14 11:40 CEST` into the context.
3. Under 10 minutes, it does nothing.

Claude sees the injection as context. You never see it as a message.

![It works](it-works.png)

## Why 10 minutes?

Short enough that Claude stays oriented across topic changes and context compactions. Long enough that it does not waste tokens on every single message.
