# Agents

## Overview

Standalone Claude Code subagent definitions. No orchestrator and no registry: the `enforcing-coding-workflow` skill calls them by name with the `Agent` tool. Both use `model: inherit`, so they run on the model the session uses.

Each call starts with no memory, so the caller must give a full brief every time.

## Agents

- **verify-agent** - VERIFY step. Runs targeted tests, a regression check, and a quick quality check on the changed files. Returns Go or No-Go. Read-only: it never edits code.
- **spec-doc-agent** - two independent modes:
  - **CHECK** (before BUILD): checks the request against `DECISIONS.md`, the log of the user's decisions. Reports a conflict as a question for the user.
  - **DOCUMENT** (after the user approves the result): updates `AGENT_CONTEXT.md`, `CHANGELOG.md`, `PROJECT_MAP.md` and appends the user's new decisions to `DECISIONS.md`.

## Response format

```
STATUS: OK | ERROR | INCOMPLETE
AGENT: [agent-name]
SUMMARY: [one line]
DETAILS: [what was checked or done]
NEXT_ACTION: [what the caller does next]
```

- **OK** - done, continue to the next stage.
- **ERROR** - failed, go back or stop and tell the user.
- **INCOMPLETE** - blocked, a question or missing information for the user.

## Adding a new agent

1. Create a `.md` file with frontmatter: name, description, model (`inherit`), color, tools.
2. Use the response format above.
3. Call it by name from the skill that needs it.
