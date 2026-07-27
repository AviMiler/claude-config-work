---
name: bug-triage
description: >
  Enforces a disciplined sequence for fixing a reported bug: reproduce it,
  isolate the actual cause, apply the smallest fix that addresses that
  cause, then verify. Blocks the common failure mode of guessing and
  rewriting large chunks of code before the root cause is known. Use when
  the user reports something broken, unexpected, or "not working" — not for
  building new features (use enforcing-coding-workflow for that).
---

# Bug Triage

## Core Rule

**No rewrite before the cause is identified.** Fixing code you haven't
diagnosed is guessing, not fixing — it risks masking the real bug or
introducing a new one. Follow this order every time, even for bugs that
feel obvious:

```
1. REPRODUCE  →  2. ISOLATE  →  3. FIX  →  4. VERIFY
```

## When This Applies

The user reports a bug: something errors, crashes, returns wrong output,
or behaves differently than expected. Does NOT apply to building new
functionality — that's `enforcing-coding-workflow`. If a triage session
turns into "also add X while we're in here," treat X as a separate task
under that skill.

## The 4 Steps

### Step 1 — Reproduce

Before reading any code, confirm you can trigger the actual failure:

- Get the exact steps, input, or command that causes it, if the user gave
  them. If not, ask.
- Run it and observe the actual failure (error message, wrong output,
  stack trace) — don't assume you know it from the description alone.
- If you cannot reproduce it, say so explicitly and ask for more specifics
  (exact input, environment, steps) rather than guessing at a fix for a
  bug you haven't seen happen.

### Step 2 — Isolate

Narrow down to the actual cause before touching any fix:

- Use the failure signal from Step 1 (error message, stack trace, wrong
  value) to locate the specific function/file/line responsible — not the
  general area.
- If the cause isn't obvious from the signal, add the minimum diagnostic
  step needed (a targeted log, a small test, reading the one relevant
  function) — not a broad exploration of the codebase.
- State the cause in one sentence before moving on: "X happens because Y."
  If you can't state it, you haven't isolated it yet — keep narrowing.

### Step 3 — Fix

- Apply the smallest change that addresses the stated cause. No drive-by
  refactors, no rewriting surrounding code "while you're in there."
- If the real fix would require a larger change (the cause traces back to
  a design issue, not a local bug), stop and tell the user what you found
  and why a bigger change is needed — don't silently expand scope.
- This fix is a code change — once applied, it falls under
  `enforcing-coding-workflow`'s Stage 3 (Verify) before you consider it
  done: brief `verify-agent` with what broke, why, and what changed.

### Step 4 — Verify

- Re-run the exact reproduction from Step 1. Confirm the original failure
  is gone.
- Confirm nothing else broke — this is what Stage 3 (Verify) in
  `enforcing-coding-workflow` already covers; don't duplicate it manually.
- Report back in plain terms: what was broken, why, what changed, and how
  the user can confirm it themselves (see that skill's verification-steps
  guidance if present).

## Failure Handling

- If Step 1 or Step 2 stalls (can't reproduce, can't isolate), stop and
  ask the user for more information rather than guessing a fix.
- If the fix in Step 3 doesn't resolve the issue in Step 4, return to
  Step 2 — the cause was misidentified, don't patch on top of a wrong fix.
