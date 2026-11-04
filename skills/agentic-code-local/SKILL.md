---
name: agentic-code-local
description: >
  Local replacement for the unavailable anthropic-skills:agentic-code skill.
  Short quality rules and tracking-file templates for AI-written code that a
  human programmer reviews and approves. Load during BUILD of every code
  change, together with code-standards.
---

# Agentic Code (local)

The code is written by you, but a human programmer tests, reviews and approves every change. Write code a human can read and a future agent can understand without the whole codebase.

## Human in the loop
- The user decides. You propose, build, verify, then wait for the user's approval.
- Never commit, push, open a PR or merge.
- If a request is unclear, ask. Do not guess.

## Code rules
- One line at the top of each new file: what it is responsible for.
- Explicit imports. No wildcards.
- Full types on function inputs and outputs, where the language supports it.
- Do not hide behavior behind more than 2 files.
- Follow the folder and naming pattern that already exists in the project.
- No unused code, no commented-out code, no abstraction used in only one place.
- At every place you change, add a very short comment: what you did and why (one line).
- When you change or remove behavior, update every doc that described the old behavior.

## Tracking files (create small, only when needed)
- `AGENT_CONTEXT.md` - read first at the start of a task. Sections: `Current State`, `Last Changes`, `Next Steps`. Keep it under 40 lines.
- `CHANGELOG.md` - one line per change: date, what, why.
- `PROJECT_MAP.md` - one line per file: path and its purpose. Update when files are added, removed or renamed.
- `DECISIONS.md` - the user's decisions. One line each:
  `D-NNN | YYYY-MM-DD HH:MM | status | decision | reason`
  Status: `received`, `done`, or `superseded by D-NNN`. Get the time with `date`.

If `PROJECT_MAP.md` exists, use it before searching the codebase. If it is wrong, fix it.
