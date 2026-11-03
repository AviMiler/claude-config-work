# Skills

See the [repo root README](../README.md) for how these fit together with `agents/`.

| Skill | Description |
|---|---|
| [`enforcing-coding-workflow`](./enforcing-coding-workflow/SKILL.md) | Workflow for every code change: start (branch) -> check -> build -> verify (with regression) -> user test -> document |
| [`agentic-code-local`](./agentic-code-local/SKILL.md) | Local replacement for `anthropic-skills:agentic-code`: short quality rules, tracking-file templates, human-in-the-loop rule |
| [`code-standards`](./code-standards/SKILL.md) | **General** code-writing rules for every stack. Loaded during BUILD. |
| [`chrome-extension-dev`](./chrome-extension-dev/SKILL.md) | **Topic-specific.** Vanilla JavaScript Chrome Extension (Manifest V3), no build tools. Loaded only for Chrome extension tasks. |
| [`new-project-setup`](./new-project-setup/SKILL.md) | Bootstraps a new project: local permissions, central branch line in the project `CLAUDE.md`, and the workflow skill |

## Convention

Every dev skill starts with a line telling the agent to also load `agentic-code-local` (and `code-standards`):

```
Read First - also load agentic-code-local and code-standards.
```

### General vs. topic-specific skills

`code-standards` is **general** and applies to every code change. **Topic-specific skills** (`chrome-extension-dev` today) hold rules for one stack and load only when the task matches. Each topic skill is its own flat folder `skills/<name>/`.

**On conflict, the topic-specific skill wins.** If it is still unclear, the agent asks the user.

A topic skill follows this skeleton (see `chrome-extension-dev/SKILL.md`):

```
Read First line
## Core Constraints          - hard "never/always" rules for this stack
## Project Structure          - expected file/folder layout
## Critical Rules / Patterns  - ready-to-use patterns, common mistakes
## Gotchas                    - non-obvious failures
## Setup Checklist            - questions to answer before writing code
```

### Project-level mandatory skills

A project can force skills to load: add a `## Required Skills` section to that project's `CLAUDE.md` listing skill names. The workflow loads them in addition to the automatic matches.
