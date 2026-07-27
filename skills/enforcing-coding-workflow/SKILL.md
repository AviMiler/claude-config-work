---
name: enforcing-coding-workflow
description: >
  Enforces a mandatory 5-stage sequence (spec-validate → build → verify → 
  document → git) for every coding task in every project. Spec-driven approach:
  validates all work against project spec, updates spec during development as
  requirements emerge, asks user to confirm new requirements before adding to
  spec, and verifies all work aligns with spec without regression. Applies
  whenever a request results in code changes. Use when the user asks to build,
  add, implement, fix, or refactor code.
---

# Coding Workflow

⚠️ Read First — also load `anthropic-skills:agentic-code` for tracking-file
templates and code quality rules used in Stage 4.

## Core Rule

Every coding task goes through all 5 stages, in order, every time. **The spec
is the single source of truth** — all work must align with it, and the spec
must be kept up-to-date as requirements emerge. Never skip a stage or the spec
check.

```
1. SPEC-VALIDATE  →  2. BUILD  →  3. VERIFY  →  4. DOCUMENT  →  5. GIT
```

If a stage fails, stop and resolve it before moving on. The spec may change
during development (Stage 2); when it does, return to Stage 1 to re-validate.

## Before Anything Else — Session Brief

At the start of any coding task in a project, before exploring the codebase:

- If `AGENT_CONTEXT.md` exists at the project root, read it first. It has
  "Last Changes", "Current State", and "Next Steps" — use it to orient
  instead of re-discovering project state via Grep/Glob/reading files.
- If it doesn't exist or looks stale (contradicts what you find while
  working), proceed normally but flag it — it should get created/refreshed
  in Stage 4.

## Explore Efficiently — Map Before You Grep

Before doing broad, exploratory codebase searches (Grep/Glob across many
files, reading files "just to see what's there"):

- Check `PROJECT_MAP.md` first if it exists — it has a one-line purpose per
  file. Use it to jump straight to the relevant file(s) instead of scanning.
- Only fall back to broad exploration when the map doesn't cover what you
  need, or doesn't exist yet.
- This applies to the main agent and to any subagent it briefs — a good
  brief (see Stage 3) is what lets a subagent skip exploration entirely.

## When This Applies

Any request that changes code: features, bug fixes, refactors, config or
dependency changes — including ones that sound small ("rename this variable,"
"fix this typo").

Does NOT apply to: pure Q&A, read-only code review, exploration, or
non-code tasks (standalone doc writing, etc.).

## The 5 Stages

### Stage 1 — Spec-Validate

**The spec is authoritative.** Check the request against it:

- If a spec file exists (spec.md, requirements.md, ARCHITECTURE.md), read it
  fully before responding. This is mandatory.
- If no spec file exists, create one (use existing docs like README,
  AGENT_CONTEXT.md, ARCHITECTURE.md as the de facto spec, then write it down).
- Confirm the request aligns with the spec. If unclear:
  - **Ask the user:** "Does this requirement belong in the spec as a permanent
    requirement, or is it a one-time exception?"
  - If permanent → update the spec document with the user's confirmation
  - If exception → proceed with a clear note that this breaks the spec
  - If conflict → stop and ask how to resolve

**Output of Stage 1:** State what will be built, which spec requirements
apply, and any new requirements that need spec approval before moving to
Stage 2.

### Stage 2 — Build

Implement the change:

- Follow the code quality rules from `anthropic-skills:agentic-code` (file size,
  naming, typing, error handling, no dead abstractions).
- Keep the change scoped to what Stage 1 validated — no drive-by refactors.
- **If new requirements emerge during build:**
  - Pause and ask the user: "A new requirement came up: [X]. Should this be
    added to the spec as a permanent requirement?"
  - If yes → update the spec, then return to Stage 1 to re-validate against
    the updated spec
  - If no → proceed with a note that this is temporary/one-off
- Track every file created, modified, or deleted for Stage 3.

**Output of Stage 2:** Working code that matches the spec, plus a list of
changed files.

### Stage 3 — Verify

**Delegate to `verify-agent`.** Run this stage via the `Agent` tool with
`subagent_type: verify-agent` (defined at
`~/.claude/agents/agents/verify-agent.md`) — a fresh subagent with its own
context window — rather than inline. If `verify-agent` is ever unavailable,
fall back to `subagent_type: general-purpose` with the same brief. This
keeps verification judgment independent of the build context and keeps
test output/logs out of the main conversation.

**Brief it fully, upfront** — the diff/files touched and what changed in
each, the reasoning behind the change, the spec requirements involved, and
what Stage 1/2 concluded. The point of a complete brief is that the agent
should not need to re-explore the codebase just to figure out what
happened; that would duplicate work the main agent already did. Its job:

1. **Judge scope first, from the brief alone.** Based on the size/risk of
   the change as described in the brief — not by re-reading the code to
   rediscover it — decide what verification is actually warranted before
   doing anything. Don't default to "run everything." A one-line typo fix
   needs a glance, not a full suite. Only once it knows what it needs to
   check should it touch the codebase — to run tests or inspect the actual
   spots relevant to that check.
2. **Run tests when they apply.** If the project has a test suite/runner (or
   the change touches testable logic), attempt to run the relevant tests —
   prefer a targeted run (affected files/modules) over a full-repo run when
   feasible. If the change is trivial (docs, comments, non-runtime config),
   skip test execution and say why.
3. **Regression check against spec:** for each relevant spec requirement,
   confirm it's still present, not degraded (performance, security,
   behavior), and not contradicted by the new code. If spec was updated in
   Stage 2, verify the new code aligns with the updated spec.
4. **Code quality budget:** on the files actually touched, check file size
   (flag files that grew unreasonably large), no new abstraction introduced
   for a single use site, no dead code left behind, naming/typing consistent
   with the rest of the file. This is a check on the diff, not a full-file
   re-review.
5. **Stay economical.** No extra tooling installs, no broad exploratory
   scans, no speculative checks beyond what the change's scope justifies.
6. **Ask before anything big.** If real verification would need something
   expensive or risky (full test suite that takes a long time, spinning up a
   dev server, hitting external services, installing dependencies, running
   migrations), the agent must stop short of doing it and report back what
   it wants to run and why — do not act on its own. Surface this to the user
   and wait for their approval before resuming the agent to finish the check.

**Output of Stage 3:** The agent reports back Go/no-go — what it checked,
what it ran (or skipped and why), and confirmation the build is safe and
spec-aligned. If no-go, return to Stage 2.

**Spec-drift audits (on-demand, outside the per-change flow).** When the
user explicitly asks for a spec audit (not tied to a specific change),
brief `verify-agent` for a **full-repo** check instead of a diff-scoped one:
give it the spec file(s) and ask it to compare the codebase against every
requirement, flagging anything that silently drifted. This is the same
agent and protocol as the regular Stage 3 check — only the brief changes
(whole repo vs. one diff) and it should only be run when asked, not as
part of every coding task (it's expensive).

### Stage 4 — Document

Update the project's tracking files per `anthropic-skills:agentic-code`'s
templates:

```
[ ] AGENT_CONTEXT.md — "Last Changes", "Current State", "Next Steps"
[ ] PROJECT_MAP.md   — any files added/removed/renamed
[ ] CHANGELOG.md     — one line per logical change with date
[ ] spec.md (or similar) — updated requirements/decisions if changed in Stage 2
[ ] ARCHITECTURE.md  — if a pattern or decision changed
[ ] DEPENDENCIES.md  — if a package was added or removed
[ ] DECISIONS.md     — only if a real choice was made between alternatives (see below)
```

If these files don't exist yet, create them from `anthropic-skills:agentic-code`'s
templates. If time is short, update `AGENT_CONTEXT.md` and the spec at
minimum.

**DECISIONS.md — record the "why", not the "what".** Add an entry only when
this task involved picking one approach over a real alternative (a library
choice, a pattern choice, a tradeoff). Skip it for straightforward
implementations with no real alternative considered — CHANGELOG.md already
covers "what changed". Format, newest entry on top:

```
## [YYYY-MM-DD] [Short title of the decision]
**Decision:** [what was chosen]
**Alternatives considered:** [what else was on the table]
**Why:** [the reasoning — constraints, tradeoffs, what tipped it]
```

Create the file (with this format as its only template, no separate
`docs/templates/` entry needed) the first time a real decision comes up.

### Stage 5 — Git

- Check for `git-instructions.md` at the project root; if present, follow it
  exactly.
- If absent, ask the user for their git workflow (branching, commit message
  style, auto-push) and write `git-instructions.md` with the answer.
- Stage only the files touched by this task (code + Stage 4 docs) — never a
  blanket `git add -A`.
- Include spec changes in the same commit (spec is code's sibling, not separate).
- Follow the system's destructive-action rules: never force-push, never skip
  hooks, confirm before pushing unless pre-authorized.

## Failure & Restart Handling

- If any stage fails or blocks, stop immediately and report which stage and why.
- Ask the user: retry this stage / adjust the spec (and re-validate) / abort.
- **If spec changes during development (Stage 2), restart from Stage 1** to
  re-validate all requirements against the updated spec before continuing.
- Never drop a stage or the spec check to save time — the user can only
  authorize exceptions explicitly.
