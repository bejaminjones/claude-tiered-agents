# claude-tiered-agents

A Claude Code plugin that lets your main session hand work to the cheapest model that can do it well. A file search goes to Haiku. A gnarly refactor goes to Opus. An architecture call goes to Fable.

It ships two things:

1. **Four subagents**, one per model tier.
2. **Routing rules** that tell Claude when to use each one. A session-start hook loads them into every session, so you don't paste anything into your `CLAUDE.md`.

## What's in the box

| Agent | Model | Effort | Use for |
|-------|-------|--------|---------|
| `claude-tiered-agents:quick-task` | Haiku | medium | File lookups, search, reading, doc lookups, quick Q&A, status checks, trivial edits |
| `claude-tiered-agents:standard-task` | Sonnet | high | Code edits, tests, refactoring, clear-spec implementations, docs |
| `claude-tiered-agents:deep-task` | Opus | medium | Complex debugging, multi-file refactors, code review, security, perf |
| `claude-tiered-agents:architect-task` | Fable | high | Architecture, the hardest debugging, high-stakes tradeoffs |

The full routing rules are in [`rules/delegation.md`](rules/delegation.md). Beyond "pick the cheapest tier that works", they cover how to brief a subagent, how to verify what comes back, when to escalate, and when to use a Workflow instead of dispatching agents by hand.

## Install

In a Claude Code session:

```
/plugin marketplace add bejaminjones/claude-tiered-agents
/plugin install claude-tiered-agents@tiered-agents-marketplace
```

Then start a new session (or run `/clear`) so the routing rules load.

### Get updates automatically

Auto-update is off by default for marketplaces you add yourself. To turn it on, open `/plugin`, go to **Marketplaces**, select `tiered-agents-marketplace`, and choose **Enable auto-update**.

To update by hand instead:

```
/plugin marketplace update tiered-agents-marketplace
```

## Requirements

- **Model access.** `architect-task` runs on Fable, `deep-task` on Opus. If your plan doesn't include a model, that agent won't run. Edit its `model:` line in a local copy, or ask Claude to use the next tier down.
- **macOS or Linux.** The session-start hook uses `cat`.

## Turn it off

`/plugin` → **Installed** → `claude-tiered-agents` → **Disable**. This removes both the agents and the routing rules.

## License

MIT
