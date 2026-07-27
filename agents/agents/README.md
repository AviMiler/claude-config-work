# Agents

## Overview

This folder holds standalone Claude Code subagent definitions. There is no
orchestrator and no registry — agents here are invoked directly by name via
the `Agent` tool, from whichever skill or conversation needs them.

## Agents

- **verify-agent** — Stage 3 (Verify) of the `enforcing-coding-workflow`
  skill. Invoked directly by that skill after Stage 2 (Build), before
  Stage 4 (Document). Judges Go/No-Go on a change: scopes what verification
  is warranted, runs targeted tests when applicable, checks for spec
  regressions, and asks before anything expensive or risky.

## Structured Response Protocol

Agents in this folder return responses in this format:

```
STATUS: OK | ERROR | INCOMPLETE
AGENT: [agent-name]
SUMMARY: [what was done]
DETAILS: [relevant output]
NEXT_ACTION: [what the caller should do next]
```

### STATUS values

- **OK** — Completed successfully, caller proceeds to the next stage
- **ERROR** — Failed, caller should stop and get developer input
- **INCOMPLETE** — Blocked on something that needs developer approval first

## Adding a New Agent

1. Create a `.md` file in `agents/` with frontmatter (name, description,
   model, color, tools).
2. Implement the structured response protocol in the agent's system prompt.
3. Reference it by name from whichever skill or workflow needs it — no
   registry file to update.

## Git Instructions

Git operations for this repo follow `git-instructions.md` at the project
root.
