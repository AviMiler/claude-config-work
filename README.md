# claude-config

Personal Claude Code configuration — skills, agents, and settings, shared
across machines via this repo. This file describes the system as a whole;
each piece documents its own details where it lives.

## What's here

```
claude-config/
├── skills/
│   └── enforcing-coding-workflow/  — the 5-stage process for every coding task
├── agents/
│   └── agents/
│       ├── spec-doc-agent.md       — Stage 1 + Stage 4 of the workflow
│       └── verify-agent.md         — Stage 3 of the workflow
├── git-instructions.md             — how this repo itself is committed/pushed
├── settings.json                   — global Claude Code settings (hooks, permissions)
└── CLAUDE.md                       — global behavioral instructions
```

## How it fits together

Every coding task (in any project, not just this one) goes through
`enforcing-coding-workflow`:

```
1. Spec-Validate → 2. Build → 3. Verify → 4. Document → 5. Git
```

Two stages are delegated to standalone subagents instead of running inline,
so their judgment stays independent of the main conversation and their
output doesn't bloat it:

- **Stage 1 + Stage 4 → `spec-doc-agent`**, invoked *twice* as the same
  resumed instance (via `SendMessage`) — Stage 1 scopes the task against
  the spec, Stage 4 (resumed later) compares what actually shipped against
  that original scope and updates the project's tracking docs.
- **Stage 3 → `verify-agent`**, a fresh subagent per task, judging Go/No-Go
  on the build: right-sized testing, spec-regression check, code-quality
  budget. Also supports an on-demand full-repo spec-drift audit, run only
  when explicitly asked.

Stages 2 and 5 (Build, Git) run inline in the main conversation — no
subagent, since they don't benefit from independent judgment the way
verification and spec/doc work do.

Full detail on each stage — the doc-sync rules, the ARCHITECTURE.md
template, the agent-legibility code rules — lives in
[`skills/enforcing-coding-workflow/SKILL.md`](skills/enforcing-coding-workflow/SKILL.md),
not duplicated here.

## Using this repo

This repo *is* `~/.claude/agents` + `~/.claude/skills` + `~/.claude/settings.json`
+ `~/CLAUDE.md`, kept in sync manually (copy + commit + push) rather than
symlinked, since Claude Code holds an open file handle on the skills/agents
folders while running.

```bash
git clone https://github.com/AviMiler/claude-config ~/claude-config
```

To pull an update from GitHub into the live `~/.claude` folders, copy the
relevant subfolder(s) over and restart Claude Code if agent/skill
definitions changed. To push a local change, copy back into this clone,
commit, and push per `git-instructions.md`.
