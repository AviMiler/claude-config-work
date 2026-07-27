# Agent Orchestration Framework

## Overview

This framework coordinates specialized Claude Code agents in a sequential pipeline to automate development workflows. Each agent has a single responsibility and communicates through a standardized protocol.

## Pipeline

```
spec-validator → builder → doc-agent → [git operations]
```

1. **spec-validator** — Validates the task against the project specification before any code is written
2. **builder** — Implements the code changes based on the validated spec
3. **doc-agent** — Updates project documentation (AGENT_CONTEXT, PROJECT_MAP, CHANGELOG, ARCHITECTURE)
4. **git-agent** — Commits and pushes changes per the project's `git-instructions.md`

## Structured Response Protocol

Every agent MUST return responses in this format:

```
STATUS: OK | ERROR | INCOMPLETE
AGENT: [agent-name]
SUMMARY: [what was done]
DETAILS: [relevant output]
NEXT_ACTION: [what the orchestrator should do next]
```

### STATUS values

- **OK** — Completed successfully, proceed to next stage
- **ERROR** — Failed, pipeline should stop for developer input
- **INCOMPLETE** — Partially done, developer should decide how to proceed

## Adding a New Agent

1. Create a `.md` file in `agents/` with frontmatter (name, description, model, color)
2. Implement the structured response protocol in the agent's system prompt
3. Register the agent in `agents/registry.md` with order, name, path, and purpose
4. If inserting mid-pipeline, update `agents/orchestrator-controller.md` sequence

## Git Instructions

Git operations are configured via `git-instructions.md` at the project root. This file tells the orchestrator how to handle commits and pushes. If the file doesn't exist when the pipeline runs, the orchestrator will ask the developer for instructions and create it.

## Invoking the Orchestrator

Provide a task description to the orchestrator-controller agent. It will read the registry, run agents in sequence, and handle errors with developer input.
