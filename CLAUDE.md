# Claude Code Instructions

Short rules. Follow them exactly.

- Every code change (feature, bug fix, refactor): load the skill `enforcing-coding-workflow` BEFORE writing any code.
- Work on the current branch. Do not ask about branches and do not switch or create branches unless the user asks.
- Never commit, push, open a PR or merge. The user does all of that.
- After build and verify, STOP. Wait for the user to test and give feedback. Only move on when the user says the result is good.
- Decisions made by the user are logged in `DECISIONS.md` in the project root, right when they are made.
- Keep answers and files short. If you are unsure, ask the user.
