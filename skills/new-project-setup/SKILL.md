---
name: new-project-setup
description: >
  Bootstraps a brand-new project folder or fresh repo checkout: detects the
  connected git remote, writes a .claude/settings.local.json with a curated
  permission set for that folder (routine file + git operations auto-allowed,
  push and destructive commands left to ask) so the agent stops prompting for
  permission on every routine action, and wires the project to use the
  enforcing-coding-workflow skill for every coding task from that point on.
  Use when the user asks to start a new project, open a new project, set up a
  new repo, or scaffold a fresh codebase.
---

# New Project Setup

⚠️ Read First — this skill only covers project *bootstrap*. The moment any
code is written, hand off to `enforcing-coding-workflow`
(`/enforcing-coding-workflow`) — every coding task in this project must go
through its stages (check → build → verify → user test → document).
Load that skill now and re-load it at the start of every future coding task
in this project; that skill in turn references `agentic-code-local`.

## When this applies

A new or near-empty folder, "start a new project," "open a new project,"
"set up a new repo," or the first session in a fresh clone.

## Steps

1. **Identify the repo.**
   - `git remote -v` — capture the connected origin URL, if any.
   - If there is no git repo yet, ask the user before running `git init` —
     creating a repo is a durable action, confirm first even during setup.
2. **Write `.claude/settings.local.json`** (create `.claude/` if missing)
   using the template below, with the path placeholders swapped for this
   project's actual absolute path. This file is personal/local — make sure
   it's git-ignored (add `.claude/settings.local.json` to `.gitignore` if not
   already covered).
3. **Point at the workflow skill.** Tell the user once that
   `enforcing-coding-workflow` now governs all coding work in this project,
   so they know why later requests will go through its stages.
4. **Never bypass confirmation during setup itself** — even though the
   template below marks some commands `ask` rather than `deny`, run nothing
   that pushes, force-modifies, or deletes without asking the user directly,
   regardless of what the JSON permits.

## Permission template

Scope every path-based rule to this project's folder — never write a bare
`Read(**)`/`Edit(**)` that would apply repo-wide across unrelated projects.

```json
{
  "permissions": {
    "allow": [
      "Read(//ABS/PATH/TO/PROJECT/**)",
      "Edit(//ABS/PATH/TO/PROJECT/**)",
      "Write(//ABS/PATH/TO/PROJECT/**)",
      "Glob(//ABS/PATH/TO/PROJECT/**)",
      "Grep(//ABS/PATH/TO/PROJECT/**)",
      "Bash(git status:*)",
      "Bash(git diff:*)",
      "Bash(git log:*)",
      "Bash(git show:*)",
      "Bash(git branch:*)",
      "Bash(git remote -v)",
      "Bash(git fetch:*)",
      "Bash(git pull:*)",
      "Bash(ls:*)",
      "Bash(mkdir:*)",
      "Bash(cat:*)"
    ],
    "ask": [
      "Bash(git checkout:*)",
      "Bash(git switch:*)",
      "Bash(git stash:*)",
      "Bash(git commit:*)",
      "Bash(git push:*)",
      "Bash(git push --force:*)",
      "Bash(git reset --hard:*)",
      "Bash(git clean:*)",
      "Bash(git rebase:*)",
      "Bash(rm -rf:*)"
    ],
    "deny": []
  }
}
```

Replace `//ABS/PATH/TO/PROJECT` with the project's real absolute path (Claude
Code's path-scoped permission syntax: `Read(//abs/path/**)`).
