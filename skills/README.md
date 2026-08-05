# Skills

See the [repo root README](../README.md) for how these fit together with `agents/`.

| Skill | Description |
|---|---|
| [`enforcing-coding-workflow`](./enforcing-coding-workflow/SKILL.md) | Mandatory 5-stage sequence for every coding task — spec-validate → build → verify → document → git |
| [`code-standards`](./code-standards/SKILL.md) | **General** code-writing rules on top of (not instead of) `anthropic-skills:agentic-code` — grows one rule at a time. Loaded during Stage 2 (Build) for every task, regardless of stack. |
| [`chrome-extension-dev`](./chrome-extension-dev/SKILL.md) | **Topic-specific.** Vanilla JavaScript Chrome Extension (Manifest V3) development — no build tools. Loaded during Stage 2 only when the task is a Chrome extension. |
| [`new-project-setup`](./new-project-setup/SKILL.md) | Bootstraps a new project — writes `.claude/settings.local.json` permissions and wires in `enforcing-coding-workflow` |

## Convention

Every dev skill starts with a reference to the official `anthropic-skills:agentic-code`
marketplace skill for shared tracking-file templates and code quality rules:

```
⚠️ Read First — also load anthropic-skills:agentic-code
```

### General vs. topic-specific skills

`code-standards` is **general** — it applies to every coding task
regardless of language or platform, and always loads in Stage 2 (Build) of
`enforcing-coding-workflow`.

**Topic-specific skills** (`chrome-extension-dev` today; a future
`react-dev`, `csharp-dev`, etc. would be the same kind of thing) hold rules
for one stack/platform only, and load in Stage 2 *only when the task
matches that stack* — `enforcing-coding-workflow` searches the available
skills for a match before Stage 2 starts. They are not nested under
`code-standards` — Claude Code only discovers one skill per top-level
`skills/<name>/` folder, so each topic skill is its own flat directory,
related to `code-standards` only by convention (its Read-First line) and
by both being active during the same stage.

**On conflict, the topic-specific skill wins** — it usually encodes a real
platform constraint the general rule couldn't have anticipated. If a
conflict can't be resolved even with that precedence, the agent asks the
user rather than picking silently (see `enforcing-coding-workflow` Stage 2).

**A topic-specific skill stays a single, self-contained file** — it does
not need to reproduce `code-standards`' whole growable-rule structure
(`### N. Title`, append-only). It follows this shared skeleton instead
(see `chrome-extension-dev/SKILL.md` as the reference example):

```
⚠️ Read First — also load anthropic-skills:agentic-code and code-standards.
[One line: where this skill wins on conflict, if relevant.]

## Core Constraints          — hard "never/always" rules for this stack
## Project Structure          — expected file/folder layout
## Critical Rules / Patterns  — ready-to-use patterns, common mistakes
## Gotchas                    — non-obvious failures specific to this stack
## Setup Checklist            — questions to answer before writing code
```

Sections are free-form beyond that skeleton — add whatever the stack
actually needs (e.g. `chrome-extension-dev`'s "System Permissions" section
is specific to this user's environment and doesn't need to appear in
every topic skill).

### Project-level mandatory skills

A project can force certain skills to load regardless of automatic
matching: add a `## Required Skills` section to that project's `CLAUDE.md`
listing skill names. `enforcing-coding-workflow` Stage 2 checks for this
and loads them unconditionally, on top of whatever it auto-matched.
