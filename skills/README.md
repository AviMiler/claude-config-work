# Skills

See the [repo root README](../README.md) for how these fit together with `agents/`.

| Skill | Description |
|---|---|
| [`enforcing-coding-workflow`](./enforcing-coding-workflow/SKILL.md) | Mandatory 5-stage sequence for every coding task — spec-validate → build → verify → document → git |
| [`bug-triage`](./bug-triage/SKILL.md) | Separate process for a reported bug — reproduce → isolate → fix → verify. Not a stage of the workflow above. |
| [`code-standards`](./code-standards/SKILL.md) | Personal code-writing rules on top of (not instead of) `anthropic-skills:agentic-code` — grows one rule at a time. Loaded during Stage 2 (Build). |
| [`chrome-extension-dev`](./chrome-extension-dev/SKILL.md) | Vanilla JavaScript Chrome Extension (Manifest V3) development — no build tools |
| [`new-project-setup`](./new-project-setup/SKILL.md) | Bootstraps a new project — writes `.claude/settings.local.json` permissions and wires in `enforcing-coding-workflow` |

## Convention

Every dev skill starts with a reference to the official `anthropic-skills:agentic-code`
marketplace skill for shared tracking-file templates and code quality rules:

```
⚠️ Read First — also load anthropic-skills:agentic-code
```
