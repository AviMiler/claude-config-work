# claude-config (work)

Claude Code configuration for the work environment: a local model with a small context window, driven by a human programmer who tests and approves every change. Everything here is short and explicit on purpose.

## What's here

```
claude-config-work/
├── skills/
│   ├── enforcing-coding-workflow/  - the workflow for every code change
│   ├── agentic-code-local/         - quality rules, file organization, tracking-file templates
│   ├── chrome-extension-dev/       - Chrome extension (MV3) rules
│   └── new-project-setup/          - bootstrap a new project
├── agents/
│   ├── spec-doc-agent.md           - CHECK against DECISIONS.md, DOCUMENT after approval
│   └── verify-agent.md             - tests + regression check, Go/No-Go
├── git-instructions.md             - user does commit/push/PR
├── settings.json                   - permissions (ask on risky git/rm commands)
└── CLAUDE.md                       - short global rules
```

## How it works

Every code change goes through `enforcing-coding-workflow`:

```
1. CHECK -> 2. BUILD -> 3. VERIFY -> 4. USER TEST -> 5. DOCUMENT
```

**Fast track** for trivial changes (typo, comment, UI string): BUILD -> short VERIFY -> USER TEST. No CHECK and no DOCUMENT.

- **CHECK**: `spec-doc-agent` compares the request with `DECISIONS.md`.
- **BUILD**: the agent writes the change.
- **VERIFY**: `verify-agent` runs tests and a regression check. Answers Go, Conditional Go (no tests exist, manual checks listed) or No-Go.
- **USER TEST**: the agent stops. The user tests and gives feedback. The loop BUILD -> VERIFY -> USER TEST repeats until the user says the result is good.
- **DOCUMENT**: `spec-doc-agent` updates the docs and marks the cycle's decisions `done`.

The user commits, pushes and opens the PR. The agent never does.

### Tracking files

Created per project, only when needed: `AGENT_CONTEXT.md` (current state), `CHANGELOG.md` (one line per change), `PROJECT_MAP.md` (one line per file). DOCUMENT updates them. Templates are in `agentic-code-local`.

### DECISIONS.md

Lives in each project's root. It holds only decisions made by the human user, one line each: `D-NNN | YYYY-MM-DD HH:MM | status | decision | reason`. A decision is written the moment the user makes it (status `received`); DOCUMENT sets it to `done`. A replaced decision gets `superseded by D-NNN`. CHECK ignores superseded lines.

## Using this repo

This repo maps to `~/.claude/agents`, `~/.claude/skills`, `~/.claude/settings.json` and `~/.claude/CLAUDE.md`. Copy the folders over and restart Claude Code if skills or agents changed. Changes to this repo itself follow `git-instructions.md`.
