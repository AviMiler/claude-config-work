# Git Instructions

Git work happens only at the START of a task. At the end of a task the user commits, pushes and opens the PR.

## At the start of every task

1. Ask the user: **new branch, or stay on the current branch?**
2. If new branch:
   - Find the central branch name in the project's `CLAUDE.md` (`Central branch: <name>`). If missing, ask the user.
   - Branch from the latest central branch: `git switch <central>`, `git pull`, then `git switch -c <new-branch>`.
3. If staying: do not switch or create branches.

## Rules

- Every branch starts from the central branch and goes back to it. No branches off other branches.
- The agent never commits, pushes, opens a PR or merges.
- Never force-push.
- Never auto-resolve merge conflicts. Report them to the user.
