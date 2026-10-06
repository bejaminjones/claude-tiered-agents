# tiered-agents

A plugin for Claude Code and Cowork that lets your main session hand work to the cheapest model that can do it well. A file search goes to Haiku. A gnarly refactor, or a careful read of conflicting reports, goes to Opus. An architecture or strategy call goes to Fable.

It ships two things:

1. **Four subagents**, one per model tier.
2. **Routing rules** that tell Claude when to use each one. A session-start hook loads them into every session, so you don't paste anything into your `CLAUDE.md`.

## What's in the box

| Agent | Model | Effort | Use for |
|-------|-------|--------|---------|
| `tiered-agents:quick-task` | Haiku | medium | File and folder lookups, grep/search, reading documents and web pages, pulling out facts, doc lookups, quick Q&A, status checks, trivial one-liner edits |
| `tiered-agents:standard-task` | Sonnet | high | Drafting documents/emails/reports, summaries, proofreading, spreadsheets, routine data work, research with a clear question, code edits, tests, refactoring, clear-spec implementations, docs |
| `tiered-agents:deep-task` | Opus | medium | Complex debugging, multi-file refactors, code review, security, perf, analysis of sources that disagree, data analysis whose figures feed a decision or get published, thorough document review — the default for heavy reasoning |
| `tiered-agents:architect-task` | Fable | high | Architecture, strategy, hardest debugging, high-stakes tradeoffs — ceiling-level, when deep-task isn't enough |

The full routing rules are in [`rules/delegation.md`](rules/delegation.md). Beyond "pick the cheapest tier that works", they cover how to brief a subagent, how to verify what comes back, when to escalate, and when to use a Workflow instead of dispatching agents by hand.

## Install

> **Renamed from `claude-tiered-agents`.** If you installed it under the old name, remove that copy first (Claude Code: `/plugin uninstall claude-tiered-agents@tiered-agents-marketplace`; Cowork: **Customize** → **Plugins** → `claude-tiered-agents` → **Remove**), then install `tiered-agents` below. The old copy no longer gets updates, and keeping both loads the rules twice.

### Claude Code

In a Claude Code session:

```
/plugin marketplace add bejaminjones/claude-tiered-agents
/plugin install tiered-agents@tiered-agents-marketplace
```

Then start a new session (or run `/clear`) so the routing rules load.

#### Get updates automatically

Auto-update is off by default for marketplaces you add yourself. To turn it on, open `/plugin`, go to **Marketplaces**, select `tiered-agents-marketplace`, and choose **Enable auto-update**.

To update by hand instead:

```
/plugin marketplace update tiered-agents-marketplace
```

### Cowork

1. Open **Customize** in the sidebar, then **Plugins**.
2. Select **Add**, then **Add marketplace**, and enter `bejaminjones/claude-tiered-agents`.
3. Install **tiered-agents**.
4. Start a new task so the routing rules load.

To get updates, select **Check for updates** on the marketplace, or turn on **Sync automatically**.

A plugin you install in Cowork is saved to your claude.ai account. It also reaches Claude Code the next time you start a session signed in to that account (Claude Code v2.1.273 or later; run `/reload-plugins` if it hasn't appeared), so you don't need to install it twice. This only works one way: a plugin installed from Claude Code's command line doesn't reach Cowork.

## Requirements

- **Model access.** `architect-task` runs on Fable, `deep-task` on Opus. If your plan doesn't include a model, that agent won't run. Edit its `model:` line in a local copy, or ask Claude to use the next tier down.
- **macOS or Linux.** The session-start hook uses `cat`.

## Turn it off

**Claude Code:** `/plugin` → **Installed** → `tiered-agents` → **Disable**.

**Cowork:** **Customize** → **Plugins** → `tiered-agents` → **Disable plugin**.

Each switch turns off both the agents and the routing rules, but only for the copy installed there. If you installed it in both Claude Code and Cowork, turn it off in both.

## License

MIT
