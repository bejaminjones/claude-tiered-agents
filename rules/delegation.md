## Model-Tiered Delegation

These rules are loaded by the `tierwise` plugin. Delegate to the cheapest model that handles the work well. The plugin provides four agents:

| Agent | Model | Effort | Use for |
|-------|-------|--------|---------|
| `tierwise:quick-task` | Haiku | medium | File and folder lookups, grep/search, reading documents and web pages, pulling out facts, doc lookups, quick Q&A, status checks, trivial one-liner edits |
| `tierwise:standard-task` | Sonnet | high | Drafting documents/emails/reports, summaries, proofreading, spreadsheets, routine data work, research with a clear question, scripted batch lookups, code edits, tests, refactoring, clear-spec implementations, docs |
| `tierwise:deep-task` | Opus | medium | Complex debugging, multi-file refactors, code review, security, perf, analysis of sources that disagree, data analysis whose figures feed a decision or get published, thorough document review — the default for heavy reasoning |
| `tierwise:architect-task` | Fable | high | Architecture, strategy, hardest debugging, high-stakes tradeoffs — ceiling-level, when deep-task isn't enough |

The short names below (`quick-task`, `deep-task`, …) mean these agents.

**Soft defaults — use judgment.** A "simple" search needing context might warrant Sonnet; a refactor in security-sensitive code might warrant Opus. Route on actual complexity, not category. **When in doubt, go one tier up** rather than risk a bad result from a model that's too cheap.

**deep-task vs architect-task.** Use `deep-task` when work is bounded to one system or one clear question, even if deep within that scope (gnarly refactor, perf hotspot, thorough review). Use `architect-task` when many interacting constraints must be held simultaneously or reasoning crosses system boundaries — architecture, concurrency, trust boundaries.

**The main thread routes.** It assesses complexity and decides what to delegate. Don't delegate the routing itself.

**Never delegate understanding.** A handoff prompt must prove you did the thinking — file paths, line numbers or the exact section, sheet or source, what to change and why. If you can't write that prompt, you don't understand the problem yet. Brief subagents like a colleague who just walked in: the goal, what you've already ruled out, and enough surrounding context to make judgment calls.

**Require evidence, not summaries.** Subagents return file:line references, command output, or screenshots — or, for knowledge work, quoted passages with their source, cell or range references, links, and paths to saved files — not prose describing what they did. A summary describes intent, not result.

**Verify what comes back.** Pair any delegated implementation with a fresh-context verifier that reads the actual diff and rendered/measured output — never the implementer's summary. Verification must anchor to ground truth outside the model (tests, screenshots, bytes on disk; for documents and data, the saved file reopened, key figures recomputed, quotes checked against their source); agents agreeing with each other doesn't count — they share the same blind spots. Frame verifiers to refute, not confirm — "try to refute this finding; default to refuted if uncertain." When something can fail in more than one way, give each verifier a distinct lens (correctness, security, does-it-reproduce) rather than identical duplicates — and check the verifier can actually reach the ground truth in question: a claim about tool or runtime behavior can't be refuted by reading files.

**Follow up, don't re-brief.** Spawn agents with a `name` so they stay addressable. For same-tier follow-ups ("you missed X, also check Y"), continue the existing agent via SendMessage — it already holds the context. Dispatch fresh only when the tier changes (escalation) or fresh eyes are the point (verification).

**Escalate with context.** When a lower-tier agent reports the task needs escalating, re-dispatch one tier up with its partial findings attached — don't make the higher tier start cold.

**Learn from routing mistakes.** One mismatch (a tier too weak for the job, or the top tier spent on something trivial) is noise. A pattern seen more than once — a tier repeatedly failing at or coasting through a task shape — belongs in memory, where the session has it (in Claude Code, an auto-memory `feedback` entry), so it shapes routing in every future session.

**Parallelize independent tasks.** Mix tiers — a Haiku search and a Sonnet edit can run simultaneously. Sequence only when one result informs the next. Agents run in the background by default and notify on completion, so dispatch freely; force synchronous (`run_in_background: false`) only when the next step needs the result.

**Script repetitive work.** When a job means many similar steps — a dozen searches, the same edit across many files, a batch of API or web lookups — write one script (shell, Python, `gh`, `curl`) that runs them, combines and de-duplicates the results, and returns only what's needed. Called one at a time, each step is a round trip and its full output lands in context. A script reaches only what a shell reaches: files, git, command-line tools, HTTP. Connected services (MCP tools such as Figma, Slack or Drive) still go call by call, or through a Workflow. Writing the script is `standard-task` work: route "many lookups, combined" there, not to `quick-task`, which is for single lookups — smaller models script less reliably.

**Fan out reads, bottleneck writes.** Read-only work (search, review, research, audits) parallelizes freely — the worst case is wasted tokens. Write work stays deliberately narrow: few agents, worktree isolation for code in a git repo (`isolation: "worktree"` on the Agent tool — built in, no manual setup), fresh-context review, and a final whole-picture pass by the main thread — integration-seam bugs are invisible to every agent inside the fan-out. Brief that final pass with the seams, not just "the whole thing": each implementer ends its report by listing what its part assumes a neighbouring part has already handled (input validation, auth, error handling, data shape). The final reviewer then traces one real request end to end across those hand-offs and checks each assumption where it lands. An assumption that no part actually enforces is the gap to report. For fan-out-shaped jobs (would mean dispatching ~5+ agents by hand: audits, migrations, adversarially-verified reviews), use a Workflow instead of manual dispatch when the Workflow tool is available — see below.

### Workflows

The Workflow tool needs per-use opt-in, but the opt-in is phrasing: "use a workflow" or "ultracode" in the request authorizes it. Absent that, propose it — don't run it.

**Tier inside workflows too.** `agent()` accepts `agentType`, `model`, and `effort` per call, so the delegation table applies stage-by-stage: cheap finders (`quick-task`, low effort) feeding expensive judges (`deep-task`, high effort). Route each stage, not the whole workflow — a ten-agent sweep priced at Haiku is a different proposition from one at Opus.

**Save recurring shapes as named workflows.** Fan-out patterns that recur (adversarial audit, migration sweep) belong in the project folder's `.claude/workflows/` as parameterized scripts — written once, invoked by name with `args`. Names register at session start, not on file creation — a script added mid-session runs via `scriptPath` instead (every inline run persists its script and reports that path, so iterate by editing it). In scripts, parse `args` defensively: it can arrive JSON-stringified rather than as an object.

**Workflows survive interruption; manual dispatch doesn't.** Every run has a `runId`; relaunching with `resumeFromRunId` returns completed agents from cache and re-runs only what hadn't finished. For long multi-agent jobs that risk hitting usage limits mid-flight, that alone justifies a Workflow even below the ~5-agent threshold.

**Prefer `pipeline()` over `parallel()`.** A barrier between stages wastes wall-clock — let each item's verification start the moment its finder returns. Barrier only when a stage genuinely needs all prior results at once (dedup across findings, early-exit on zero).
