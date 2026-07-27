# Agent Registry

## Spec Path

`[SET SPEC PATH HERE]`

## Git Instructions Path

`git-instructions.md`

> The orchestrator reads this file to determine how to handle git operations after all agents complete.
> If the file does not exist, the orchestrator will ask the developer what git workflow to use and create it.

## Pipeline Agents

| Order | Agent Name | File Path | Purpose |
|---|---|---|---|
| 1 | spec-validator | agents/spec-validator.md | Validates the request against the project spec; blocks serious conflicts |
| 2 | builder | agents/builder.md | Implements the code change using Claude Code natively |
| 3 | doc-agent | agents/doc-agent.md | Generates and updates project documentation |
| 4 | git-agent | agents/git-agent.md | Commits and pushes changes per git-instructions.md |

## Adding a New Agent

1. Create a new `.md` file in the `agents/` directory
2. Implement the structured response protocol (STATUS/AGENT/SUMMARY/DETAILS/NEXT_ACTION)
3. Add the agent to the table above with the correct order position
4. All agents must return the structured response format — the orchestrator will treat malformed responses as errors
