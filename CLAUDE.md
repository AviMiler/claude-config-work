# Claude Code Instructions

Short rules. Follow them exactly.

- Every code change (feature, bug fix, refactor): load the skill `enforcing-coding-workflow` BEFORE writing any code.
- Ask the user at the start of every task: new branch or stay on the current one? A new branch always starts from the central branch.
- The central branch of a project is written in that project's own `CLAUDE.md` as `Central branch: <name>`. If the line is missing, ask the user and offer to add it.
- Never commit, push, open a PR or merge. The user does all of that.
- After build and verify, STOP. Wait for the user to test and give feedback. Only move on when the user says the result is good.
- Decisions made by the user are logged in `DECISIONS.md` in the project root.
- Keep answers and files short. If you are unsure, ask the user.
