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

⚠️ Read First — also load `anthropic-skills:agentic-code` (tracking-file
templates, general code quality) and `code-standards` (personal rules on
top of it). Load a topic-specific skill too if one matches this task's
stack (see Stage 2).

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

At the start of any coding task, before exploring the codebase: if
`AGENT_CONTEXT.md` exists at the project root, read it first ("Last
Changes"/"Current State"/"Next Steps") instead of re-discovering project
state via Grep/Glob. If missing or stale, proceed but flag it for Stage 4.

## Explore Efficiently — Map Before You Grep

Before broad exploratory searches, check `PROJECT_MAP.md` first if it
exists and jump straight to the relevant file(s). Only fall back to broad
exploration when the map doesn't cover what you need. **If what you find
contradicts the map** — stop trusting it for this task, fix the stale
entry immediately, then continue; a stale map is worse than no map. This
applies to subagents you brief too — a complete brief (Stage 3) is what
lets a subagent skip exploration entirely.

## When This Applies

Any request that changes code — features, bug fixes, refactors, config or
dependency changes — including small ones ("fix this typo").

Does NOT apply to: pure Q&A, read-only code review, exploration, or
non-code tasks.

## The 5 Stages

### Stage 1 — Spec-Validate

**Delegate to `spec-doc-agent`** (`~/.claude/agents/agents/spec-doc-agent.md`,
runs on Sonnet — its judgment here gates everything downstream) — this is
**Invocation 1**. Brief it with the task request and spec file path(s) if
known. Full decision logic (when it asks the user vs. proceeds) lives in
that agent's own file — don't duplicate it here.

**⚠️ Save the returned agent id/name immediately.** Stage 4 must resume
this *exact* instance via `SendMessage`, not spawn a fresh one — hold onto
the id/name across Stages 2-3, or the Stage 4 handoff breaks.

If `spec-doc-agent` is unavailable, do Stage 1 inline with the same logic.

**Output:** what will be built, which spec requirements apply, any new
requirement pending spec approval — before moving to Stage 2.

### Stage 2 — Build

Implement the change, scoped to what Stage 1 validated — no drive-by
refactors:

- Follow `anthropic-skills:agentic-code` + `code-standards`. **Before
  writing code, also search available skills for a topic-specific match**
  (e.g. `chrome-extension-dev`) by stack/framework/platform and load it if
  one applies — zero, one, or several, don't force it. Also check the
  project's `CLAUDE.md` for a `## Required Skills` section and load those
  unconditionally.
- **On conflict, the topic-specific skill wins** over general rules. If
  that precedence doesn't cleanly resolve it, ask the user rather than
  picking silently.
- Full detail on writing agent-legible code and keeping docs honest on
  *changes* (not just additions) is in
  [`references/build-rules.md`](references/build-rules.md) — read it now.
- **If new requirements emerge:** ask the user whether to add it to the
  spec permanently (→ update spec, return to Stage 1) or treat it as a
  one-off (→ proceed with a note).
- Track every file created/modified/deleted for Stage 3.

**Output:** working code matching the spec, plus the list of changed files.

### Stage 3 — Verify

**Delegate to `verify-agent`** (`~/.claude/agents/agents/verify-agent.md`) —
a fresh subagent, not inline, so verification stays independent and test
output stays out of the main conversation. Fall back to `general-purpose`
if unavailable.

**Brief it fully** — diff/files touched, reasoning, spec requirements
involved, Stage 1/2's conclusions — so it doesn't need to re-explore. Its
own file defines the full checklist (scope judgment, targeted tests,
spec-regression check, code-quality budget, economy, ask-before-anything-big)
— don't duplicate that logic here.

**Output:** Go/no-go, what it checked/ran/skipped and why. No-go → back to
Stage 2.

**On-demand spec-drift audits:** when explicitly asked (not tied to one
change), brief `verify-agent` for a full-repo check against the spec
instead of a diff — same agent, different scope. Only on request, never
automatic (expensive).

### Stage 4 — Document

**Resume `spec-doc-agent`** via `SendMessage` with the id/name from Stage 1
— this is **Invocation 2** of the same instance, not a fresh spawn. Brief
it with what Stage 2 actually did and Stage 3's verdict; flag explicitly
if the outcome deviated from Stage 1's plan. If the id/name was lost, say
so explicitly — the agent will refuse to silently reconstruct Stage 1 and
returns `STATUS: INCOMPLETE`; gather what Stage 1 concluded and supply it,
and tell the user the handoff broke.

Default: **don't touch a file unless its condition is met**, and skip
silently (no explanation) when it isn't. The exact per-file checklist
(which conditions gate AGENT_CONTEXT.md, CHANGELOG.md, PROJECT_MAP.md,
spec.md, ARCHITECTURE.md, DEPENDENCIES.md, DECISIONS.md) lives in
`spec-doc-agent.md`'s "INVOCATION 2 — Document" section — single source,
don't copy it here.

If `spec-doc-agent` is unavailable, read that section of its file directly
and do Stage 4 inline with the same checklist and deviation-flagging rule.

### Stage 5 — Git

- Follow `git-instructions.md` at the project root if present; if absent,
  ask the user for their workflow and write it.
- Stage only the files this task touched (code + Stage 4 docs) — never
  `git add -A`. Spec changes ride in the same commit as the code.
- Never force-push, never skip hooks, confirm before pushing unless
  pre-authorized.

## Failure & Restart Handling

- Any stage fails/blocks → stop, report which stage and why, ask the user:
  retry / adjust spec and re-validate / abort.
- Spec changes during Stage 2 → restart from Stage 1. If `spec-doc-agent`
  is already running for this task, **resume that instance** (still before
  Invocation 2) — don't spawn a second one, or Stage 4 won't know which to resume.
- Never drop a stage or the spec check to save time — only the user can
  authorize an exception, explicitly.
