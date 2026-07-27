# Git Instructions

## Branch

Always commit directly to `main`. Do not create feature branches, develop branches, or any other branches.

## Workflow

1. Before committing, pull latest from remote: `git pull origin main`
2. Stage only the files that were changed (never `git add -A`)
3. Commit with a conventional commit message derived from what Stage 2 (Build) changed
4. Push to `main`: `git push origin main`

## Rules

- No branching — all work happens on `main`
- Always sync local with the GitHub remote before and after committing
- Never force-push
- Never auto-resolve merge conflicts — report them as errors
