# Cowork support — design

**Date:** 2026-10-06
**Status:** Draft, awaiting review

## Goal

Make `claude-tiered-agents` a good fit for Claude Cowork users as well as Claude Code users. It should be one public plugin, published for anyone, that routes both office work and coding to the cheapest capable model.

## What we already know

- Anthropic's platform-support page says Cowork loads `agents/*.md` and `hooks/hooks.json` from plugins. Chat ignores both. Cowork installs from **Customize → Plugins → Add marketplace** (GitHub `owner/repo` works) or by uploading a zip. The manifest format is the same as Claude Code's. Source: https://claude.com/docs/plugins/platform-support
- A live test in a Cowork task (2026-10-06), with the current plugin installed from the marketplace, showed:
  - the SessionStart hook ran and the routing rules were in context;
  - all four agents were listed with the correct model and effort;
  - `quick-task` reported running on Haiku 4.5;
  - the Workflow tool is available in Cowork.
- A plugin installed in Cowork syncs to Claude Code on the same account. A second, Cowork-only plugin would therefore double-load rules for people who use both. **Decision: one plugin, no fork.**

## Problem

Nothing breaks in Cowork. The problem is wording. The agent descriptions and the rules assume software work: code edits, tests, file:line evidence, worktrees. A Cowork user doing research, documents or spreadsheets gets routing guidance that doesn't describe their tasks. So the main thread under-uses or mis-routes the tiers.

## Changes

### 1. Agent files (`agents/*.md`)

The `description` frontmatter is what the main thread routes on. Widen each one, and its "You excel at" list, to cover knowledge work beside code. Keep `name`, `model`, `effort` and `color` unchanged.

| Agent | Add (examples) |
|---|---|
| quick-task | finding and reading files and folders, pulling facts from documents or web pages, quick lookups, simple reformatting |
| standard-task | drafting documents, emails and reports; summarising; building or editing spreadsheets; routine data cleanup and analysis; research with a clear question |
| deep-task | careful analysis of long or conflicting sources, data analysis where errors matter, thorough review of a document or plan |
| architect-task | strategy and high-stakes decisions with many interacting constraints, beside the existing architecture/security cases |

The body text becomes domain-neutral where it says "code" generically. For example, "Follow existing code patterns" becomes "Follow the existing conventions of the project, document or codebase". Code-specific bullets stay.

### 2. Rules (`rules/delegation.md`)

- **Table "Use for" column:** add the office-work examples above to each row.
- **Never delegate understanding:** "file paths, line numbers" becomes "file paths, line numbers, the exact section, sheet or source".
- **Require evidence:** add the knowledge-work forms: quoted passages with their source, cell or range references, links, and paths to saved files.
- **Verify what comes back:** add knowledge-work ground truth: open the saved file and check what's actually in it, recompute key figures, check quotes against the source.
- **Fan out reads, bottleneck writes:** mark worktree isolation as "for code in a git repo". The rest of the rule applies as is.
- **Learn from routing mistakes:** "auto-memory" becomes "memory, where the session has it". Cowork's memory support isn't confirmed.
- **Workflows section:** keep it. It was confirmed available in Cowork. `.claude/workflows/` becomes "the project folder's `.claude/workflows/`".

No new rules. No removed rules.

### 3. README

- Opening line: "A plugin for Claude Code and Cowork".
- Split **Install** into *Claude Code* (unchanged) and *Cowork* (Customize → Plugins → Add marketplace → `bejaminjones/claude-tiered-agents` → install → start a new task).
- Note that a Cowork install also reaches Claude Code on the same account, so there's no need to install twice.
- **Turn it off:** add the Cowork path (open the plugin → Disable plugin).
- Update the table's "Use for" column to match the rules.

### 4. Manifests

Mention Cowork in the `description` in `plugin.json` and `marketplace.json`. Change the marketplace `description` from "for Claude Code" to "for Claude Code and Cowork". Add `cowork` to `keywords`.

## Out of scope

- Submitting to Anthropic's plugin directory. That's a separate, public decision for Ben after this lands.
- Windows support. The hook uses `cat`. Cowork's support on Windows is untested, so the README's existing macOS/Linux requirement stays.
- Renaming agents or changing models or effort.

## Testing

1. In Claude Code: `claude --plugin-dir .` in a fresh session. Confirm the rules load and the four agents are listed. That shows the change hasn't regressed Claude Code.
2. Push, then in Cowork run **Check for updates** on the marketplace. Start a new task and repeat the 2026-10-06 test prompt.
3. Routing check in Cowork: give one knowledge-work task, for example "find the three largest files in my Downloads folder and draft a one-paragraph summary of the biggest one". Confirm the main thread delegates the lookup to `quick-task` and the drafting to `standard-task` or does it itself, rather than using `deep-task` for everything.
4. A fresh-context reviewer reads the final diff against this spec.
