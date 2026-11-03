---
name: enforcing-coding-workflow
description: >
  Mandatory workflow for every code change in every project: ask about the
  branch, check the request against DECISIONS.md, build, verify (including a
  regression check), wait for the user's testing and feedback, then document.
  Use whenever a request will change code: build, add, implement, fix or
  refactor.
---

# Coding Workflow

Read first: load `agentic-code-local` and `code-standards`. If the task matches a topic skill (for example `chrome-extension-dev`), load it too. If the project `CLAUDE.md` has a `## Required Skills` section, load those skills.

Applies to ANY change to code, even a small one. Does NOT apply to questions, read-only review or exploration.

## The stages, in order

```
0. START  ->  1. CHECK  ->  2. BUILD  ->  3. VERIFY  ->  4. USER TEST  ->  5. DOCUMENT
```

Do not skip a stage. If a stage fails, stop and tell the user.

### 0. START (git, only here)
Ask the user: new branch or stay on the current one? Follow `git-instructions.md`. A new branch always starts from the central branch named in the project `CLAUDE.md`.

### 1. CHECK
Call the `spec-doc-agent` in CHECK mode. Brief it with: the user's request and the path of `DECISIONS.md`. It reports if the request conflicts with a past user decision. If there is a conflict, ask the user what to do before building.

### 2. BUILD
- Do only what the request asks. No extra refactoring.
- Follow `agentic-code-local` and `code-standards`.
- Remember every file you create, change or delete.

### 3. VERIFY
Call the `verify-agent`. Brief it with: the request, the list of changed files, what you changed and why. It runs targeted tests and a **regression check** (does everything that worked before still work?). It answers Go or No-Go.
- No-Go: go back to BUILD and fix.
- Go: continue to USER TEST.

### 4. USER TEST (stop here)
STOP. Tell the user what changed and how to test it. Wait for the user's testing and feedback.
- The user reports a problem: go back to BUILD, then VERIFY, then USER TEST again.
- Repeat until the user says the result is good.
- Do NOT go to DOCUMENT before the user approves.

### 5. DOCUMENT
Call the `spec-doc-agent` in DOCUMENT mode. Brief it with: what was built, the verify verdict, which files changed, and any decision the user made during this cycle. It updates the project docs and adds a line to `DECISIONS.md` if the user made a decision.

After DOCUMENT, tell the user it is done. The user does commit, push and PR.

## Decisions log
After every cycle (BUILD, VERIFY, USER TEST), if the user made a decision, it must be written to `DECISIONS.md`. Only decisions made by the user, not your own choices.

## If an agent is unavailable
Do its job yourself in the same way, using the steps in the agent file.
