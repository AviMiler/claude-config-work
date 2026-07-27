# claude-skills

Personal Claude skill library — shared across claude.ai and Claude Code.

## Skills

| Skill | Description |
|---|---|
| [`enforcing-coding-workflow`](./enforcing-coding-workflow/SKILL.md) | Mandatory 4-stage sequence for every coding task — validate → build → document → git |
| [`chrome-extension-dev`](./chrome-extension-dev/SKILL.md) | Vanilla JavaScript Chrome Extension (Manifest V3) development — no build tools |
| [`new-project-setup`](./new-project-setup/SKILL.md) | Bootstraps a new project — writes `.claude/settings.local.json` permissions and wires in `enforcing-coding-workflow` |

## Structure

```
claude-skills/
├── enforcing-coding-workflow/
│   └── SKILL.md
├── chrome-extension-dev/
│   ├── SKILL.md
│   └── references/
│       └── chrome-extension.md
├── new-project-setup/
│   └── SKILL.md
└── README.md
```

## Usage in Claude Code

```bash
# Clone once
git clone https://github.com/YOUR_USERNAME/claude-skills ~/.claude-skills

# Update
git -C ~/.claude-skills pull
```

## Convention

Every dev skill starts with a reference to the official `anthropic-skills:agentic-code`
marketplace skill (this repo no longer ships its own copy):

```
⚠️ Read First — load anthropic-skills:agentic-code skill
```
