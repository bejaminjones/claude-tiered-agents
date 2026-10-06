# Cowork Support Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Reword the plugin's agents, rules, README and manifests so it serves Cowork's knowledge work as well as Claude Code's coding, as one plugin.

**Architecture:** No structural change. Cowork already loads `agents/*.md` and the SessionStart hook (confirmed in a live Cowork test on 2026-10-06). Every change is text: agent `description` frontmatter and body, `rules/delegation.md`, `README.md`, and the two manifests.

**Tech Stack:** Markdown with YAML frontmatter, JSON manifests, Claude Code plugin format.

**Spec:** `docs/superpowers/specs/2026-10-06-cowork-support-design.md`

## Global Constraints

- One plugin, no fork. Plugin name stays `claude-tiered-agents`. Marketplace name stays `tiered-agents-marketplace`.
- Agent frontmatter `name`, `model`, `effort` and `color` must not change: quick-task haiku/medium, standard-task sonnet/high, deep-task opus/medium, architect-task fable/high.
- No rules added or removed in `rules/delegation.md`. Wording changes only.
- `hooks/hooks.json` is untouched.
- The README keeps the macOS/Linux requirement. Windows is out of scope.
- No submission to Anthropic's directory. No push to `main` without Ben's go-ahead.

## Review Focus

1. **Model or effort silently changed while editing frontmatter.** A user would get a different, possibly pricier, model. Pinned by the frontmatter diff check in Task 4, Step 2.
2. **Coding routing regresses in Claude Code.** Widened descriptions shouldn't pull code work away from the tiers it went to before. Pinned by the Claude Code live check in Task 4, Step 4.
3. **Broken markdown in the hook output.** For example, a table row with a stray `|` would garble the rules every session. Pinned by the rendered-table check in Task 4, Step 3.
4. **Cowork install steps don't match the real UI labels.** A stranger following the README gets stuck. Pinned by Ben's Cowork run in Task 5, which follows the README word for word.
5. **Knowledge work in Cowork all goes to deep-task, or all stays on the main thread.** Pinned by the routing prompt in Task 5.

---

### Task 1: Widen the four agent files

**Files:**
- Modify: `agents/quick-task.md`
- Modify: `agents/standard-task.md`
- Modify: `agents/deep-task.md`
- Modify: `agents/architect-task.md`

**Interfaces:**
- Produces: agent descriptions whose "Use for" wording Task 2's table and Task 3's README table mirror.

- [ ] **Step 1: quick-task.** Replace the `description:` line with:

```
description: Fast, cheap agent for simple tasks — file and folder lookups, searches, reading documents and web pages, pulling out specific facts, quick checks, simple Q&A, status checks, simple reformatting, and other lightweight operations that don't require deep reasoning.
```

Replace the "You excel at" list with:

```
You excel at:
- Finding and reading files, folders, documents and web pages
- Searching codebases (grep, glob, file reads)
- Pulling specific facts, figures or quotes out of a source
- Answering quick factual questions about code or documents
- Checking file contents, git status, build output
- Simple one-liner edits, fixes or reformatting
```

Leave the opening sentence and the Guidelines unchanged.

- [ ] **Step 2: standard-task.** Replace the `description:` line with:

```
description: Everyday workhorse agent for routine work — drafting documents, emails and reports, summarising, building or editing spreadsheets, routine data cleanup and analysis, research with a clear question, writing code, implementing features, writing tests, refactoring, running builds, documentation, and other standard work that needs competence but not deep reasoning.
```

Replace the body (everything after the frontmatter) with:

```
You are a capable assistant for everyday work in documents, spreadsheets and code. You handle the bread-and-butter tasks efficiently and correctly.

You excel at:
- Drafting documents, emails and reports from a clear brief
- Summarising sources into what the reader needs
- Building and editing spreadsheets; routine data cleanup and analysis
- Research that answers a clear question, with sources
- Implementing features from clear specifications
- Writing and updating tests
- Refactoring code for clarity or performance
- Writing documentation and comments
- Running builds and interpreting output
- Standard bug fixes with clear reproduction steps
- File creation and project scaffolding

Guidelines:
- Follow the existing conventions of the project, document or codebase.
- Get it right on the first pass.
- If you encounter something that requires architectural or strategic judgment, complex debugging, or careful weighing of conflicting sources, say so and suggest escalating rather than making assumptions.
- Check your work where possible before reporting completion — run the tests, reopen the saved file, recompute key figures.
```

- [ ] **Step 3: deep-task.** Replace the `description:` line with:

```
description: Heavy-duty agent for complex work requiring deep reasoning — complex debugging, multi-file refactors, code review, security analysis, performance optimization, careful analysis of long or conflicting sources, data analysis where errors matter, and thorough review of a document or plan. The default for heavy reasoning work; reach for architect-task only when the stakes warrant maximum effort.
```

In the body:
- The first sentence becomes: `You are an expert-level assistant for complex, high-stakes work. You bring deep reasoning, careful analysis, and thorough consideration to problems where precision matters.`
- Append three bullets to "You excel at":

```
- Careful analysis of long or conflicting sources
- Data analysis where errors matter
- Thorough review of a document, plan or proposal
```

- The review guideline becomes: `- When reviewing, be thorough but focus on what matters — for code, correctness, security, and maintainability; for documents, accuracy and soundness of reasoning — over style.`

- [ ] **Step 4: architect-task.** Replace the `description:` line with:

```
description: Ceiling-level reasoning agent for the hardest problems — architectural and strategic decisions with long-lasting consequences, debugging that has resisted other approaches, subtle security analysis, complex tradeoff evaluation with many interacting constraints. Reserve for tasks where deep-task would not be enough and where the stakes justify maximum reasoning effort.
```

Append to "You excel at":

```
- Strategy and high-stakes decisions with many interacting constraints, where a wrong call is expensive to undo
```

Leave everything else unchanged.

- [ ] **Step 5: Verify frontmatter untouched.**

Run: `git diff -- agents | grep -E '^[-+](name|model|effort|color):'`
Expected: no output.

- [ ] **Step 6: Commit.**

```bash
git add agents
git commit -m "Widen agent descriptions to cover knowledge work for Cowork"
```

### Task 2: Reword the routing rules

**Files:**
- Modify: `rules/delegation.md`

**Interfaces:**
- Consumes: the Task 1 descriptions (the table mirrors them).
- Produces: the "Use for" column that Task 3's README table copies verbatim.

- [ ] **Step 1: Table "Use for" column.** Replace the four rows' last cells with:

| Agent | Use for |
|---|---|
| quick-task | `File and folder lookups, grep/search, reading documents and web pages, pulling out facts, doc lookups, quick Q&A, status checks, trivial one-liner edits` |
| standard-task | `Drafting documents/emails/reports, summaries, spreadsheets, routine data work, research with a clear question, code edits, tests, refactoring, clear-spec implementations, docs` |
| deep-task | `Complex debugging, multi-file refactors, code review, security, perf, analysis of long or conflicting sources, data analysis where errors matter, thorough document review — the default for heavy reasoning` |
| architect-task | `Architecture, strategy, hardest debugging, high-stakes tradeoffs — ceiling-level, when deep-task isn't enough` |

- [ ] **Step 2: Never delegate understanding.** `file paths, line numbers, what to change and why` → `file paths, line numbers or the exact section, sheet or source, what to change and why`.

- [ ] **Step 3: Require evidence.** Replace the second sentence with: `Subagents return file:line references, command output, or screenshots — or, for knowledge work, quoted passages with their source, cell or range references, links, and paths to saved files — not prose describing what they did.`

- [ ] **Step 4: Verify what comes back.** `(tests, screenshots, bytes on disk)` → `(tests, screenshots, bytes on disk; for documents and data, the saved file reopened, key figures recomputed, quotes checked against their source)`.

- [ ] **Step 5: Learn from routing mistakes.** `belongs in auto-memory as a \`feedback\` entry` → `belongs in memory (as an auto-memory \`feedback\` entry, where the session has auto-memory)`.

- [ ] **Step 6: Fan out reads.** `worktree isolation (\`isolation: "worktree"\` on the Agent tool — built in, no manual setup)` → `worktree isolation for code in a git repo (\`isolation: "worktree"\` on the Agent tool — built in, no manual setup)`.

- [ ] **Step 7: Workflows.** `belong in the project's \`.claude/workflows/\`` → `belong in the project folder's \`.claude/workflows/\``.

- [ ] **Step 8: Verify the rule count is unchanged.**

Run: `git diff --stat -- rules/delegation.md && grep -c '^\*\*' rules/delegation.md`
Expected: the bold-rule count equals `git show HEAD:rules/delegation.md | grep -c '^\*\*'` (15 at the time of writing).

- [ ] **Step 9: Commit.**

```bash
git add rules/delegation.md
git commit -m "Reword routing rules so evidence and verification cover knowledge work"
```

### Task 3: README and manifests

**Files:**
- Modify: `README.md`
- Modify: `.claude-plugin/plugin.json`
- Modify: `.claude-plugin/marketplace.json`

**Interfaces:**
- Consumes: the Task 2 "Use for" column, verbatim.

- [ ] **Step 1: README opening.** The first paragraph becomes:

```
A plugin for Claude Code and Cowork that lets your main session hand work to the cheapest model that can do it well. A file search goes to Haiku. A gnarly refactor, or a careful read of conflicting reports, goes to Opus. An architecture or strategy call goes to Fable.
```

- [ ] **Step 2: README table.** Replace the four "Use for" cells with the Task 2 Step 1 text, verbatim.

- [ ] **Step 3: README install.** Under `## Install`, put the existing content under a new `### Claude Code` heading. Demote "Get updates automatically" to `####`. Then add:

```
### Cowork

1. Open **Customize** in the sidebar, then **Plugins**.
2. Select **Add**, then **Add marketplace**, and enter `bejaminjones/claude-tiered-agents`.
3. Install **claude-tiered-agents**.
4. Start a new task so the routing rules load.

To get updates, select **Check for updates** on the marketplace, or turn on **Sync automatically**.

A plugin you install in Cowork is saved to your claude.ai account. It also reaches Claude Code the next time you start a session signed in to that account, so you don't need to install it twice.
```

- [ ] **Step 4: README turn-off.** Under `## Turn it off`, keep the Claude Code line, prefixed `**Claude Code:**`. Add: `**Cowork:** **Customize** → **Plugins** → \`claude-tiered-agents\` → **Disable plugin**.`

- [ ] **Step 5: plugin.json.** Set `description` to `"Four model-tiered subagents (Haiku, Sonnet, Opus, Fable) plus routing rules that load at session start, so the main thread in Claude Code or Cowork delegates each job — code or knowledge work — to the cheapest model that handles it well."`. Append `"cowork"` to `keywords`.

- [ ] **Step 6: marketplace.json.** Set the top-level `description` to `"Model-tiered subagents for Claude Code and Cowork"`. Set the plugin entry `description` to `"Four model-tiered subagents plus routing rules that load at session start, for Claude Code and Cowork"`.

- [ ] **Step 7: Validate JSON.**

Run: `python3 -m json.tool .claude-plugin/plugin.json >/dev/null && python3 -m json.tool .claude-plugin/marketplace.json >/dev/null && echo ok`
Expected: `ok`

- [ ] **Step 8: Check the tables match.**

Run:

```bash
diff <(grep '^| .claude-tiered-agents:' rules/delegation.md | awk -F'|' '{print $5}') \
     <(grep '^| .claude-tiered-agents:' README.md | awk -F'|' '{print $5}')
```

Expected: no output.

- [ ] **Step 9: Commit.**

```bash
git add README.md .claude-plugin
git commit -m "Document Cowork install and mention Cowork in manifests"
```

### Task 4: Verify in Claude Code

**Files:** none modified.

- [ ] **Step 1: Validate the plugin.** Run `claude plugin validate .` if `claude plugin --help` lists `validate`. Otherwise skip, and note the skip in the report.
Expected: no errors.

- [ ] **Step 2: Frontmatter is unchanged against main.**

Run: `git diff main -- agents | grep -E '^[-+](name|model|effort|color):'`
Expected: no output.

- [ ] **Step 3: The hook output renders.**

Run: `CLAUDE_PLUGIN_ROOT=. sh -c 'cat "${CLAUDE_PLUGIN_ROOT}/rules/delegation.md"' | grep -c '^| `claude-tiered-agents:'`
Expected: `4`. Then confirm each of those rows has exactly 5 `|` characters.

- [ ] **Step 4: Live Claude Code session loads the local copy.**

Run: `claude -p --plugin-dir . "Quote the Use-for cell for standard-task from the delegation rules loaded at session start, then name which agent you would send each of these to and why: (a) rename a variable across three files, (b) find which file defines the login screen, (c) diagnose a race condition between two services."`
Expected: the quoted cell contains `Drafting documents/emails/reports`. Routing: (a) standard-task, (b) quick-task, (c) deep-task or architect-task. That's the same as before the change.

- [ ] **Step 5: Fresh-context review.** Dispatch a reviewer (deep-task) with the spec path and `git diff main`. Ask it to refute that the diff implements the spec, and to name any spec line with no matching change.

### Task 5: Verify in Cowork (Ben)

Gated: needs Ben's go-ahead to merge to `main` and push, because Cowork installs from the GitHub repo.

- [ ] **Step 1:** Merge the branch to `main` and push.
- [ ] **Step 2 (Ben):** In Cowork, follow the README "Cowork" section word for word. If the plugin is already installed, use **Check for updates** instead. Report any step whose label doesn't match the screen.
- [ ] **Step 3 (Ben):** Start a new task and paste: *"Quote the Use-for cell for standard-task from the delegation rules loaded at session start. Then find the three largest files in my Downloads folder and draft a one-paragraph summary of the biggest one. Say which agent did each part."*
Expected: the quote contains `Drafting documents/emails/reports`. The lookup goes to quick-task. The draft goes to standard-task or stays on the main thread, not deep-task.
