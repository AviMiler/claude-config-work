---
name: enforcing-coding-workflow
description: >
  Mandatory workflow for every code change in every project: check the request
  against DECISIONS.md, build, verify (including a regression check), wait for
  the user's testing and feedback, then document. Trivial changes use a fast
  track. Use whenever a request will change code: build, add, implement, fix or
  refactor.
---

# Coding Workflow

Read first: load `agentic-code-local`. If the task matches a topic skill (for example `chrome-extension-dev`), load it too. If the project `CLAUDE.md` has a `## Required Skills` section, load those skills.

Applies to ANY change to code. Does NOT apply to questions, read-only review or exploration.

Work on the current branch. Do not ask about branches (see `git-instructions.md`).

## Pick the track

**Fast track** - trivial change only: typo, comment, docs text, a variable name, a UI string. Not for logic changes.
```
BUILD  ->  VERIFY (short)  ->  USER TEST
```
No CHECK and no DOCUMENT. VERIFY only reads the diff. USER TEST is still mandatory. Decisions made by the user are still logged in `DECISIONS.md`. If unsure whether a change is trivial, use the full track. Tell the user which track you chose.

**Full track** - everything else.
```
1. CHECK  ->  2. BUILD  ->  3. VERIFY  ->  4. USER TEST  ->  5. DOCUMENT
```

Do not skip a stage of the track. If a stage fails, stop and tell the user.

## The stages

### 1. CHECK
Call the `spec-doc-agent` in CHECK mode. Brief it with: the user's request and the path of `DECISIONS.md`. It reports if the request conflicts with an active past user decision. If there is a conflict, ask the user what to do before building.

### 2. BUILD
- Do only what the request asks. No extra refactoring.
- Follow `agentic-code-local`.
- Remember every file you create, change or delete.

### 3. VERIFY
Call the `verify-agent`. Brief it with: the request, the list of changed files, what you changed and why. It runs targeted tests and a **regression check** (does everything that worked before still work?). It answers Go, Conditional Go or No-Go.
- No-Go: go back to BUILD and fix.
- Go: continue to USER TEST.
- Conditional Go (no tests exist): continue to USER TEST and give the user the manual checks the agent listed.

### 4. USER TEST (stop here)
STOP. Tell the user what changed and how to test it. Wait for the user's testing and feedback.
- The user reports a problem: go back to BUILD, then VERIFY, then USER TEST again.
- Repeat until the user says the result is good.
- Do NOT go to DOCUMENT before the user approves.

### 5. DOCUMENT
Call the `spec-doc-agent` in DOCUMENT mode. Brief it with: what was built, the verify verdict, which files changed, and the IDs of the decisions of this cycle. It updates the project docs and marks those decisions `done` in `DECISIONS.md`.

After DOCUMENT, tell the user it is done. The user does commit, push and PR.

## Decisions log
Record a decision in `DECISIONS.md` the moment the user makes it, at any stage, with status `received`. Do not wait for DOCUMENT. Only decisions made by the user, not your own choices.

Line format: `D-NNN | YYYY-MM-DD HH:MM | status | decision | reason`
- `D-NNN`: next free number, 3 digits.
- Time: run `date '+%Y-%m-%d %H:%M'`. Do not guess the time.
- Status: `received` when recorded. DOCUMENT changes it to `done`. When the user changes an old decision, write `superseded by D-NNN` on the old line and add the new line.
- Never delete lines.

## If an agent is unavailable
Do its job yourself in the same way, using the steps in the agent file.
