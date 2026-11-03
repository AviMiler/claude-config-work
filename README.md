# claude-config (work)

Claude Code configuration for the work environment: a local model with a small context window, driven by a human programmer who tests and approves every change. Everything here is short and explicit on purpose.

## What's here

```
claude-config-work/
├── skills/
│   ├── enforcing-coding-workflow/  - the workflow for every code change
│   ├── agentic-code-local/         - quality rules + tracking-file templates
│   ├── code-standards/             - general code rules
│   ├── chrome-extension-dev/       - Chrome extension (MV3) rules
│   └── new-project-setup/          - bootstrap a new project
├── agents/agents/
│   ├── spec-doc-agent.md           - CHECK against DECISIONS.md, DOCUMENT after approval
│   └── verify-agent.md             - tests + regression check, Go/No-Go
├── git-instructions.md             - branch question at task start; user does commit/push/PR
├── settings.json                   - permissions only
└── CLAUDE.md                       - short global rules
```

## How it works

Every code change goes through `enforcing-coding-workflow`:

```
0. START -> 1. CHECK -> 2. BUILD -> 3. VERIFY -> 4. USER TEST -> 5. DOCUMENT
```

- **START**: the agent asks whether to open a new branch or stay on the current one. A new branch always comes off the project's central branch (written in that project's `CLAUDE.md` as `Central branch: <name>`). This is the only git step.
- **CHECK**: `spec-doc-agent` compares the request with `DECISIONS.md`.
- **BUILD**: the agent writes the change.
- **VERIFY**: `verify-agent` runs tests and a regression check.
- **USER TEST**: the agent stops. The user tests and gives feedback. The loop BUILD -> VERIFY -> USER TEST repeats until the user says the result is good.
- **DOCUMENT**: `spec-doc-agent` updates the docs and logs the user's new decisions.

The user commits, pushes and opens the PR. The agent never does.

### DECISIONS.md

Lives in each project's root. It holds only decisions made by the human user, one line each: `YYYY-MM-DD HH:MM | decision | reason`. After every cycle, if the user made a decision, a line is added.

## Using this repo

This repo maps to `~/.claude/agents`, `~/.claude/skills`, `~/.claude/settings.json` and `~/.claude/CLAUDE.md`. Copy the folders over and restart Claude Code if skills or agents changed. Changes to this repo itself follow `git-instructions.md`.
